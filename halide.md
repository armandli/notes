Optimization example: 3x3 box filter

un-optimized algorithm:

```
void box_filter_3x3(const Image &in, Image &blury){
  Image blurx(in.width(), in.height());

  for (int y = 0; y < in.height(); y++)
    for (int x = 0; x < in.width(); x++)
      blurx(x, y) = (in(x-1, y) + in(x, y) + in(x+1, y)) / 3;

  for (int y = 0; y < in.height(); y++)
    for (int x = 0; x < in.width(); x++)
      blury(x, y) = (blurx(x, y-1) + blurx(x, y) + blurx(x, y+1)) / 3;
}
```

unoptimized version runs 9.96 ms per megapixel. multiple problems with the un-optimized version:
1. it uses a temporary, which when being read in the second loop, is no longer in the cache
2. first loop access pattern for in is bad, elements are not next to each other
3. no simd instructions in parallelized version

optimized version of the same code is 11 times faster

```
void fast_blur(const Image& in, Image& blurred){
  _m128i one_third = _mm_set1_epi16(21846);
  #pragma omp parallel for
  for (int yTile = 0; yTile < in.height(); yTyle += 32){
    _m128i a, b, c, sum, avg;
    _m128i tmp[(256/8)*(32+2)];
    for (int xTile = 0; xTile < in.width(); xTile += 256){
      _m128i *tmpPtr = tmp;
      for (int y = -1; y < 32+1; y++){
        const uint16_t *inPtr = &(in(xTyle, yTile+y));
        for (int x = 0; x < 256; x += 8){
          a = _mm_loadu_si128((_m128i*)(inPtr-1));
          b = _mm_loadu_si128((_m128i*)(inPtr+1));
          c = _mm_loadu_si128((_m128i*)inPtr);
          sum = _mm_add_epi16(_mm_add_epi16(a, b), c);
          avg = _mm_mulhi_epi16(sum, one_third);
          _mm_store_si128(tmpPtr++, avg);
          inPtr += 8
        }
      }
      tmpPtr = tmp;
      for (int y = 0; y < 32; ++y){
        _m128i *outPtr = (_m128i*)(&(blurred(xTile, yTile+y)));
        for (int x = 0; x < 256; x += 8){
          a = _mm_load_si128(tmpPtr+(2*256)/8);
          b = _mm_load_si128(tmpPtr+256/8);
          c = _mm_load_si128(tmpPtr++);
          sum = _mm_add_epi16(_mm_add_epi16(a, b), c);
          avg = _mm_mulhi_epi16(sum, one_third);
          _mm_store_si128(outPtr++, avg);
        }
      }
    }
  }
}
```

optimized the pipeline with parallelism and locality. parallelism using omp to distribute across threads SIMD parallel vectors. locality by making sure the image being processed at each stage is in cache in the next stage before freeing them. this way the parallelized algorithm is not hindered by memory bandwidth.

the image is computed tile by tile, keeps intermediate data in cache.

1. unoptimzied version keeps a intermediate stage of blurx, complete the stage and then move on to the next stage. the optimzied version interleaves the stages. to compute a chunk of blury, it takes a piece from input, compute the chunk's blurx, and then immediately consume the piece. and then the next chunk.
2. the optimized version may also re-do work that was previous thrown away

optimization choice space:
1. in what order should it compute its values?
   1. scan line order, y, then x (y is horizontal)
   2. reverse, do x first then y
   3. serial y, vectorize x by 4
   4. parallel y, vectorize x by 4
   5. split x by 4 and y by 4, compute each 4 by 4 tile, then compute the inner y and x
2. when should it compute its inputs?
   1. all at once, ahead of time (non-optimized version's choice). slow because no locality, by the time when intermediate data is read it is gone from cache
   2. as needed, discarding intermediate after use. this causes recomputation of the same set of data
   3. as needed, but reusing the old values to avoid the redundant work, but now introduced a serial dependency, code cannot be parallelized

these choices make a chocie space of 2 dimentions: 1 is compute granularity, 1 is storage granularity. if it is computed all at once, ahead of time, then both storage and compute granularity are course grained. if doing as needed, discarding after use, then both dimention are fine grained. if doing as needed, resuing old values, then it is doing fine grained computation, but course grained storage, making it harder to parallelize.

these choices above are the extreme ends of each choice, but the fast choices are somewhere in the middle. some middle choices:
1. split input into 4 tiles, using 2 threads, 1 does top 2 tiles, 1 does bottom 2. fully compute the blurx in one tile, then move on to produce output for each tile for each thread
2. compute output in scan lines, vectorize and paralleize within each scan line, reuse old intermediate values, and have 2 threads, doing criss-cross.
3. choice 3 is similar to 2, but divide the data into 2 rectangular zones, each thread work on one zone instead of doing the same scan line at the same time as choice 2. choice 3 does small amount of redundant work at the boundary.

choice 3 turns out to be the fastest blur3x3

### Reference
[Halide Talk](https://youtu.be/3uiEyEKji0M)
