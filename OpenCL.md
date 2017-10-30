# OpenCL Notes

### Components/Boilerplate

- Platform
- Device
- Context
- CommandQueue
- Program
- Memory
- NDRange

### Models
Platform Model
Execution Model
Memory Model

### Platform Model

- list supported platform
- version for each platform
- list of supported operations on the platform
- devices from each platform

to find out maximum number of threads per workgroup, use clGetDeviceInfo with param_name set to CL_DEVICE_MAX_WORK_GROUP_SIZE. Most compatible work group size is 16.

device memory is also limited. to query for device memory limit. use clGetDeviceInfo with CL_DEVICE_LOCAL_MEM_SIZE.

functions:

clGetPlatformIDs
clGetPlatformInfo
clGetDeviceIDs
clGetDeviceInfo

barrier(CLK_LOCAL_MEM_FENCE)

### Execution Model

dimension of the parallel execution, from 1D to 3D. Each dimension has global size, local size and number of work groups.

local size * number of work groups = global size

Context is the environment OpenCL kernel is run. A Context can contain one device or multiple devices. Kernels running on the same context should share the same CommandQueue.

A CommandQueue is created for every device. Possible to have multiple CommandQueue for different tasks.

Each command or task is associated with a event. Events can be used as synchronization mechanism to coordinate execution between host and device. There is no synchronization between different CommandQueue.

CommandQueue supports in order or out of order execution.

host functions:

clEnqueueNDRangeKernel
clCreateContext
clCreateCommandQueue
clEnqueueWriteBuffer
clEnqueueNDRangeKernel

kernel builtin functions:

get_work_dim()         # total number of dimensions of the kernel being launched
get_global_size(dim)   # global number of work item in dimension specified in dim
get_global_id(dim)     # global id in dimension dim
get_local_size(dim)    # get the size of local work group in dimension dim
get_local_id(dim)      # get the id local to a work group in dimension dim
get_num_groups(dim)    # number of work groups in dimension dim
get_group_id(dim)      # return group id of work group in dimension dim
get_global_offset(dim) # return the offset value specified in global work offset for dimension dim.
barrier()              # global or local memory fence

### Memory Model

4 memory regions:
- global memory
- constant memory
- local memory
- private memory

global memory are prefixed with keyword "global". global memory is accessible by all CU.

constant memory is initialized by hostconstant memory is initialized by host. It is equivalent to creating a buffer with CL_MEM_READ_ONLY flag.

local memory is fast and can only be shared within the same group. it is identified by the "local" keyword

### Buffer Object

It's possible to create sub-buffer out of buffers. Usage subbuffer could be dividing buffer across multiple devices and launch kernels through different CommandQueue for each buffer.

one way to create a buffer without any data transfer is use the option CL_MEM_USE_HOST_PTR. 

local memory are also sent to kernel as argument. the creation of global memory and local memory is different using clSetKernelArgs. Global memory will take a reference of the global buffer from host. whereas local memory creation pass in a null pointer.

blocking read and blocking write blocks the host device from continuing until the read and write from and to device is complete. OpenCL uses relaxed memory model, which means sme non-cache coherent device may see different values for the same global address, hense explicit synchronization is required. When blocking is not used, the host device has to use clFinish and clWaitForEvents to ensure command has been completed.

can also read/write rectangular/cubic segment. each takes additional parameter buffer_row_pitch and buffer_slice_pitch. Offset to cl_mem object is computed as:

buffer_origin[2] * buffer_slice_pitch + buffer_origin[1] * buffer_row_pitch + buffer_origin[0]

clEnqueueCopyBuffer enable application to copy data between two device buffer. It is equivalent to reading a buffer back from device to host, then write to another device buffer. If both cl_mem are within the same context, then it may not go to host.

OpenCL can map a region directly into host memory. The function is clEnqueueMapBuffer. While clEnqueueReadBuffer reads from a memory location pre-allocated. clEnqueueMapBuffer returns a pointer to the mapped region. the clEnqueueMapBuffer uses flags CL_MAP_READ, CL_MAP_WRITE. When a unmap is called, CL_MEM_READ is optimized by quickly relinquishing the data held by mapped region, whereas CL_MAP_WRITE will have to copy the region back to the device.

It is a common consensus that memory mapping gives significant improvement in performance compared to regular read/write commands. OpenCL driver can utilize DMA transfer to transfer data to host. Efficiency is dependent on OpenCL implementation.

Steps of memory mapping:
1. use clEnqueueMapBuffer to map device memory to host
2. perform operation on mapped buffer
3. unmap the mapped buffer using clEnqueueUnmapMemObject

It is possible to query for buffer objects using clGetMemObjectInfo.

Undefined behaviors in cl_mem:
1. simultaneous read and write to a buffer
2. buffer is created with CL_MEM_WRITE_ONLY and kernel reads from this buffer pointer on device.
3. buffer is created with CL_MEM_READ_ONLY and kernel write to this buffer.
4. it is possible to create cl_mem from the same host memory using CL_MEM_USE_HOST_PTR flag. There may be overlapping memory region. if one or more commands enqueued operate on the two cl_mem objects but pointing to the same host memory host_ptr, then such an operation is not defined. User must guarantee no writing and reading to the same host_ptr simultaneously.
5. similar to 4, reading from, writing and copying cl_mem and its corresponding subbuffer object is undefined
6. cl_mem object created using CL_MEM_USE_HOST_PTR should meet the requirement that they contain the last bits. i.e. simultaneous writes from the host and the device kernel is undefined.
7. Releasing a buffer before task is queued to use it (write or read).

functions:

clCreateBuffer
clCreateSubBuffer
clEnqueueWriteBuffer #write to device
clEnqueueReadBuffer  #read from device
clFinish
clWaitForEvents

clEnqueueReadBufferRect
clENqueueWriteBufferRect

clEnqueueCopyBuffer
clEnqueueCopyBufferRect

clEnqueueMapBuffer
clEnqueueUnmapMemObject

clSetKernelArgs  # kernel arguments are set using functions

### Program Objects

Application can create multiple  program objects, each for a different context. Each program object can have more than one kernel object. Kernels are identified using the keyword kernel.

Program object is created once for a context in execution. Program is created from either source text or binary. Loading from binary is much faster than loading from source. Difference is binary file could be device specific. Some implementation may store an intermediate representation. Format of binary file is implementation specific and openCL vendors are free to choose any format of representation.

Binary or not, the program must call clBuildProgram. This is because clBuildProgram acts as linker.

Program info can be retrieved using clGetProgramInfo function. Once program is built, we can get the binary file using clGetProgramInfo function with param CL_PROGRAM_BINARIES.

CL_BUILD_PROGRAM_FAILURES is returned when compiler failed to build the kernel during clBuildProgram. clGetProgramBuildInfo can be used to retrieve the build failure message.

example logging of compiler error macro:

'''
#define LOG_OCL_COMPILER_ERROR(prog, dev) \
{\
  cl_int logstat; \
  char* buildlog = NULL;\
  size_t buildlogsize = 0;\
  logstat = clGetProgramBuildInfo(prog, dev, CL_PROGRAM_BUILD_LOG, buildlogsize, buildlog, &buildlogsize);\
  if (logstat != CL_SUCCESS){ \
    std::cout << "Error # " << logstat \
              << ":: clGetProgramBuildInfo<CL_PROGRAM_BUILD_LOG> failed."; \
    exit(1); \
  }\
  buildlog = (char*) malloc(buildlogsize);\
  if (buildlog == NULL){\
    std::cout << "Faile to allocate host memory. (build log)\n"; \
    return -1; \
  }\
  memset(buildlog, 0, buildlogsize); \
  logstat = clGetProgramBuildInfo(prog, dev, CL_PROGRAM_BUILD_LOG, buildlogsize, buildlog, NULL);\
  if (logstat != CL_SUCCESS){ \
    std::cout << "Error # " << logstat \
              << ":: clGetProgramBuildInfo<CL_PROGRAM_BUILD_LOG> failed."; \
    exit(1); \
  }\
  std::cout << " \n\t\t BUILD LOG\n"; \
  std::cout << " buildLog << std::endl;\
  free(buildlog);\
}
'''

OpenCL program build options:
ENABLE_ATOMICS enables atomic support
cl-single-precision-constant treats all constants as single precision
-denorms-are-zero denormal values are truncated to 0
cl-opt-disable disables all optimizations
cl-mad-enable allows for a * b + c to be replaced by a mad operation, mad are different from fma (fused multiply add) operation, later being more precise

Information retrievable from OpenCL program:
CL_PROGRAM_BINARIES
CL_PROGRAM_REFERENCE_COUNT
CL_PROGRAM_CONTEXT
CL_PROGRAM_NUM_DEVICES
CL_PROGRAM_DEVICES
CL_PROGRAM_SOURCE
CL_PROGRAM_BINARY_SIZE
CL_PROGRAM_NUM_KERNELS #a program may be associated with many kernels
CL_PROGRAM_KERNEL_NAMES

creating binary from source:
'''
cl_program prog;
cl_int clstat = CL_SUCCESS;
cl_device_id* devlist = NULL;
prog = clCreateProgramWithSource(context, 1, (const char**)&kernel_code, NULL, &clstat);

clstat = clGetContextInfo(context, LC_CONTEXT_NUM_DEVICES, sizeof(num_devices), &num_devices, NULL);
devlist = new cl_device_id[num_devices];
clstat = clGetContextInfo(context, CL_CONTEXT_DEVICES, num_devices * sizeof(cl_device_id), devlist, NULL);
clstat = clBuildProgram(prog, num_devices, devlist, NULL, NULL, NULL);

clstat = clGetProgramInfo(prog, CL_PROGRAM_NUM_DEVICES, sizeof(cl_uint), &num_devices, &bytes_read);
size_t* binarysz = new size_t[num_devices];
clstat = clGetProgramInfo(prog, CL_PROGRAM_DEVICES, sizeof(cl_device_id) * num_devices, devlist, &bytes_read);

clstat = clGetProgramInfo(prog, CL_PROGRAM_BINARY_SIZES, sizeof(size_t) * num_devices, binarySize, &bytes_read);

char** progbin = new char*[num_devices];
for (cl_uint i = 0; i < num_devices; ++i)
  progbin[i] = new char[binarySize[i]];

clstat = clGetProgramInfo(prog, CL_PROGRAM_BINARIES, sizeof(unsigned char *) * num_devices, progbin, &bytes_read);
'''

SPIR = standard compliant binary as part of OpenCL extension

SPIR is a mapping of OpenCL C program to LLVM IR. It adopts two notions: binary bit code representation and assembly language notation provided by LLVM.

clCreateKernelsInProgram creates all kernel objects associated with the program

clEnqueueNDRangeKernel launches the kernel. It takes work_dim that represents global work offset, global work size, and local_work_size. event wait list is a list of operations that needs to be done before the execution of the kernel.

clEnqueueTask for task parallel workloads if clEnqueueNDRange has parallel workload available.

clGetKernelInfo gets information about a cl_kernel object. Information contains kernel function name, number of arguments, associated program, context etc.

clGetKernelArgInfo gets information about a kernel's argument. The argument information is only available for the program if it's created with source, not binary.

clGetKernelWorkGroupInfo gets the kernel's work group size. optimal performance can be achieved if a multiple of CL_KERNEL_PREFERRED_WORK_GROUP_SIZE_MULTIPLE is used. CL_KERNEL_PREFFERRED_WORK_GROUP_SIZE_MULTIPLE can be retrieved using clGetKernelWorkGroupInfo. Along with other information such as local memory size, private mem size, global work size, work group size.

clReleaseProgram releases the program object. clReleaseProgram will decrease a reference count, object is deleted if count reaches 0.

clRetainProgram increases the reference count on a program object. the reference count can be obtained by calling clGetKernelInfo with param CL_PROGRAM_REFERENCE_COUNT.

some kernel come with opencl library, and are considered builtin. use clCreateProgramWithBuiltInKernel to create them.

functions:

clCreateProgramWithSource
clCreateProgramWithBinary
clBuildProgram
clGetProgramInfo
clGetProgramBuildInfo
clLinkProgram
clCreateKernelsInProgram
clEnqueueTask
clGetKernelInfo
clGetKernelArgInfo
clGetKernelWorkGroupInfo
clReleaseProgram
clRetainProgram
clCreateProgramWithBuiltInKernel

### Events and Synchronization

openCL provides both coarse-grained event and fine-grained events. 

Coarsed grained events is achieved using clFlush and clFinish. Fine-grained events is achieved using cl_event objects, which determine the status of a task enqueued on a CommandQueue. 

clFinish function returns if all commands on a queue is completed. clFinish will block the host program.

clEventInfo can get the info on a cl_event object.

clWaitForEvents can be used to block host program until list of events is finished in queue.

clEnqueue* functions will automatically wait on event_wait_list before executing itself.

events can be used in 3 ways:
1. host notification
2. command synchronization
3. profiling

scenario: single device in-order usage. There is no need for fine-grained synchronization of events. clFlush and clFinish object should suffice.

scenario: single device and out-of-order queue. commands dequeued has no guarantee of order. transaction may overlap and device starts executing as soon as it can. device may have capability to execute multiple tasks simultaneously. need explicit synchronization.

scenario: multiple device and different opencl contexts. in this model command queues from different devices could not synchronize between contexts.

scenario: multiple device single opencl context. multiple devices in a platform belonging to the same context and each device has an associated queue and will modify or read data from a combined memory pool. need programming expertise to divide workload across different devices. once each device completes its execution the associated event is set to CL_COMPLETE. host program is expected to explicitly track the status of each task queue. this is coarsed-grained synchronizazation and opencl provides different functions to achieve this.

#### Coarse-grained Synchronization

uses clFlush, clFinish. all blocking commands also call clFlush. If there is an error code associated with a command event handle then the task is abnormally terminated. Error code could be CL_INVALID_COMMAND_QUEUE or CL_OUT_OF_RESOURCES or CL_OUT_OF_HOST_MEMORY etc.

clEnqueueBarrierWithWaitList queues a synchronization point. It is a non-blocking call and can achieve the same result as clFinish. Any command enqueued after the barrier will not continue its execution until barrier has reached state CL_COMPLETE. It's similar to clFinish, but is used for out-of-order queue.

#### Event based fine-grained Synchronization

event states:
CL_QUEUED
CL_SUBMITTED
CL_RUNNING
CL_COMPLETE

clEnqueueMarkerWithWaitList does not stop execution of subsequet tasks enqueued. It is used to catch the status of execution of all commands enqueued before it. Marker creates a event that subsequent operation can easily wait on instead of all the events the marker is waiting on.

clWaitForEvents is synchronous in nature.

cl_event is also an object that needs to be explicitly freed. It is also reference counted. clRetainEvent increments the reference count, while clReleaseEvent decrements the reference count.

#### User-created Event

user can create custom events using clCreateUserEvent. All user created event reaches CL_SUBMITTED first. They do not reach CL_QUEUED.

clSetUserEventStatus can change the status of the user event, otherwise it is always CL_SUCCESS.

#### Event Profiling

CommandQueue should be created with CL_QUEUE_PROFILING_ENABLE flag in order to be able to profiling based on event. clGetEventProfilingInfo contain timestamps on when a event is queued, submitted, start and end. the value is a cl_ulong, which is device time in nanoseconds.

the following code calculates the time of a event:

'''
double get_event_exec_time(cl_event ev){
  cl_ulong start, end;
  clGetEventProfilingInfo(ev, CL_PROFILING_COMMAND_START, sizeof(cl_ulong), &start, NULL);
  clGetEventProfilingInfo(ev, CL_PROFILING_COMMAND_END, sizeof(cl_ulong), &end, NULL);
  return (end - start) * 1e-6;
} 
'''

#### Memory Fences

synchronize between workgroup. No way to synchronize between different work groups. 2 types of fences;
1. barrier(CLK_LOCAL_MEM_FENCE) ensures correct ordering of oepration on local memory
2. barrier(CLK_GLOBAL_MEM_FENCE) ensures correct ordering of operatiosn on global memory

both can be used together too.

functions:

clFinish
clFlush
clGetEventInfo
clWaitForEvents
clEnqueueBarrierWithWaitList
clEnqueueMarkerWithWaitList
clRetainEevent
clReleaseEvent
clCreateUserEvent
clGetEventProfilingInfo

### OpenCL Programming

CL_DEVICE_ADDRESS_BITS is either 32 or 64. This is similar to whether size_t will be either 32 or 64 bit in application. 

intptr_t and uintptr_t allows any pointer type to convert to this type, and then be converted back to a void pointer.

It is advised to use opencl data types e.g. cl_float, cl_int, cl_uint instead of program types and make sure the sizes of these types match.

openCL kernel data alignment is either 32 or 64, which could be greater than the alignment of your C compiler host code. This means an object of size 48 bytes need a filler of 32 bytes when put on openCL kernel. Manual alignment specification for the strust is needed. Use __attribute__ (align(32)) on host to make sure alignment match.

opencl also supports vector data types. vector literals can be formed using list of scalars or vectors or mixture of scalar and vector. e.g.

int4 i4 = (int4)(1,2,3,4);
int4 i4 = {1,2,3,4};

if the literal is of the form of a single scalar, the value is replicated across all lanes. this is not allowed:

float4 f = (float4)(2.f, 1.f);

if vector type has component size 2,3,4, and components can be accessed via [x,y], [x,y,z] and [x,y,z,w]. e.g. v.x v.y v.z v.w v.xxyy v.xyzw v.wzxy

higher vectors e.g. 16 can use .sC where C is 0 to F

can also access vector via subcomponents such as .lo .hi .even .odd

it is illegal to take the address of vector sub components

OpenCL standard is based on strict aliasing rules of C99 standard. This means an alias cannot be created for the type other than the original type.

Conversion between data types are allowed, except for vector types.

explicit conversion is allowed using the form convert_dstType(srcType), e.g. int4 k = convert_int4(j); // where convert_int4 is a builtin function in openCL

convert_dstType_sat(srcType) also controls saturation of the destination type. saturation allows maximum possible value which the destination can take.

rounding mode can be specified in the built-in explicit conversion. e.g. rtc (round to nearest even), rtp (round to positie infinity), rtz (round towards zero). rtz is the default mode, for float as destination the default rounding mode is rte (round to next even). syntax for rounding specification is the same as saturation.

it's also possible to convert a non-vector type to a vector type using explicit conversion e.g. int4 k = (int4)200;

pointer aliasing is not allowed in openCL, and memcpy is not defined in openCL therefore the only way to reinterpret float binary value is through union.

openCL also provides as_type(srcType) as way to reinterpret cast a value from one to another without changing the bit pattern.

functions:

convert_dsttype(srctype)
convert_dsttype_sat(srctype)
convert_disttype_rmode(srctype)
as_type(srctype)

address space qualifiers:
global or __global
local or __local
constant or __constant
private or __private

if address space qualifier is not specified, default assumptions are made based on type. by default it is private, this include kernel arguments. pointer arguments must declare if it is global, constant or local. function pointer as argument is not allowed. outer kernel function interface cannot take argument type pointer to pointer.

static and extern are supported, register and auto is not.

math, geometric, and relational function built-ins are available in openCL specification section 6.12

#### Optimization Techniques

example of obtaining kernel execution time:

'''
cl_event ev;
cl_ulong start;
cl_ulong end;

clEnqueueNDRangeKernel(queue, kern, 2, NULL, globalThreads, localThreads, 0, NULL, &ev);
clWaitForEvents(1, &ev);
clGetEventProfilingInfo(ev, CL_PROFILING_COMMAND_START, sizeof(cl_ulong), &start, 0);
clGetEventProfilingInfo(ev, CL_PROFILING_COMMAND_END, sizeof(cl_ulong), &end, 0);
'''

coalesced memory access technique : the ith worker should access (i + k)-th element of global memory. this way memory load will be faster.
local memory: put memory that needs to be repeatedly acceessed in local memory instead of accessing through global

general tips:
1. minimize host device memory transfer, and hide memory transfer latency with parallel computation. it may even be better to re-compute something instead of copying from host.
2. one large transfer is better than multiple small transfer amounting to the same size.
3. try use coalesced memory access. avoid out-of-sequence and misaligned transactions.
4. use local memory for caching and private memory.
5. avoid bank conflict. multiple read/write on same memory bank become serialized instead of parallel. bank conflict is about local memory. local memory are accessed through limited set of gates, each gate is capable of reading only one 32bit address from a stripe of local memory. e.g. with 16 banks, starting from memory address 0, it can only be read from bank 0, then address 32 can only be read from bank 1, address 64 is read through bank 2,  ... address 512 can only be read from bank 0 etc. when multile threads in a single warp is trying to access different memory address that can only go through the same bank gate, it creates a bank conflict and the read become serialized. with warp size 16 it can go up 16 way bank conflict, making the entire warp serial. high performant cuda code will try to avoid having bank conflicts
6. number of work groups shuld always be bigger than number of compute units. more work group per compute unit hides latency.
7. number of work items per work group should be multiple of warps/wavefront size. beneficial to even have dummy work items.
8. try to increase number of occupancy which is ratio of active warp per compute unit to maximize allowed wraps per compute unit.
9. instruction throughput defined as number of instruction executed per cycle. use smaller number of cycles to get things done. avoid automatic conversion from double to float.
10. use non-blocking command queue and multiple commands before being flushed to GPU
11. avoid branch or divergent branch within a warp. minimize number of instructions within branch. e.g. 

if (cond) x += y; else x -= z;

instead do

int tmp = cond ? y : -z;
x += tmp;

avoid nexted if, and avoid if with multiple conditions combined by AND operators.

12. for 2D or 3D data, use texture and image memory, which has hardware accelerated data type conversion and interpolation optimized for 2D/3D caching.
13. prefer constant memory over global memory.
14. avoid barrier if possible
15. algorithm cascading. multiple algorithm implementing the same goal work together. this is to maximize memory throughput. the bus size on GPU are 128 bit, therefore it is faster reading 4 floats, or 2 doubles from global memory, than reading a single float or double. so locally we can do a small serial algorithm.
16. when number of threads running is smaller euqal to a single warp, then synchronization is no longer needed
17. loop unrolling will speed things up
18. cuda and openCL on GPU supports vector types of up to 128 bits. e.g. float4, double2. it is most optimal to read and write in float4 and double2 than float and double.
19. modern nvidia cards have 32 banks.
20. general macro of avoiding bank conflict:

```opencl
#define NUM_BANKS 16
#define LOG_NUM_BANKS 4
#define CONFLICT_FREE_OFFSET(n) \
  ((n) >> NUM_BANKS + (n) >> (2 * LOG_NUM_BANKS))
```

#### Reduction Example

naive implementation:

```opencl
kernel void reduce0(global double* dst, global double* src, local double* sm){
  size_t gid = get_global_id(0);
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);

  //local a block to shared memory
  sm[lid] = src[gid];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  for (size_t i = 0; i < lsize; i *= 2){
    if (lid % (2 * i) == 0){
      sm[lid] += sm[lid + i];
    }
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

problem: highly divergent warp threads. each wrap will always have threads that are either idle or doing work. this slows things down. solution is put active threads together.

```opencl
kernel void reduce1(global double* dst, global double* src, local double* sm){
  size_t gid = get_global_id(0);
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);

  //local a block to shared memory
  sm[lid] = src[gid];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  for (size_t i = 0; i < lsize; i *= 2){
    size_t idx = lid * 2 * i;
    if (idx < lsize){
      sm[idx] += sm[idx + i];
    }
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

problem: bank conflict in shared memory. solution use sequantial addressing.

```opencl
kernel void reduce2(global double* dst, global double* src, local double* sm){
  size_t gid = get_global_id(0);
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);

  //local a block to shared memory
  sm[lid] = src[gid];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  for (int i = lsize >> 1; i > 0; i >>= 1){
    if (lid < i){
      sm[lid] += sm[lid + i]
    }
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

problem: half of the threads are only used for loading data into sm, maybe we don't have to have them in the first place. solution, each thread load 2 values during initial loading.

```opencl
kernel void reduce3(global double* dst, global double* src, local double* sm){
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);
  size_t gid = group_id * lsize * 2 + lid;

  //local a block to shared memory
  sm[lid] = src[gid] + src[gid + lsize];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  for (int i = lsize >> 1; i > 0; i >>= 1){
    if (lid < i){
      sm[lid] += sm[lid + i]
    }
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

optimization: we can do better using loop unrolling. in the last iteration where the number of active threads is smaller than a warp (32 threads), we no longer need synchronization. in fact, we no longer need a test.

```opencl
kernel void reduce4(global double* dst, global double* src, local double* sm){
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);
  size_t gid = group_id * lsize * 2 + lid;

  //local a block to shared memory
  sm[lid] = src[gid] + src[gid + lsize];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  for (size_t i = lsize >> 1; i > 32; i >>= 1){
    if (lid < i){
      sm[lid] += sm[lid + i]
    }
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid < 32){
    sm[lid] += sm[lid + 32];
    sm[lid] += sm[lid + 16];
    sm[lid] += sm[lid + 8];
    sm[lid] += sm[lid + 4];
    sm[lid] += sm[lid + 2];
    sm[lid] += sm[lid + 1];
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

optimzation: we can do even better at loop unrolling, the remaining for loop can also be unrolled.

```opencl
const size_t blocksize = 512; //has to be constant
kernel void reduce5(global double* dst, global double* src, local double* sm){
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);
  size_t gid = group_id * lsize * 2 + lid;

  //local a block to shared memory
  sm[lid] = src[gid] + src[gid + lsize];
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  if (blocksize >= 512){ //these will be reduced to noop
    if (lid < 256)
      sm[lid] += sm[lid + 256];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (blocksize >= 256){
    if (lid < 128)
      sm[lid] += sm[lid + 128];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (blocksize >= 128){
    if (lid < 64)
      sm[lid] += sm[lid + 64];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid < 32){
    if (blocksize >= 64) sm[lid] += sm[lid + 32];
    if (blocksize >= 32) sm[lid] += sm[lid + 16];
    if (blocksize >= 16) sm[lid] += sm[lid + 8];
    if (blocksize >= 8)  sm[lid] += sm[lid + 4];
    if (blcoksize >= 4)  sm[lid] += sm[lid + 2];
    if (blockszie >= 2)  sm[lid] += sm[lid + 1];
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```

optimzation: algorithm cascading, we can do more work when initially loading data into local memory to speed up and utilize the memory load

```opencl
const size_t blocksize = 512; //has to be constant
kernel void reduce6(global double* dst, global double* src, local double* sm, int n){
  size_t lid = get_local_id(0);
  size_t lsize = get_local_size(0);
  size_t group_id = get_group_id(0);
  size_t gid = group_id * blocksize * 2 + lid;
  size_t gridsize = blocksize * 2 * lsize;

  //local a block to shared memory
  sm[lid] = 0.;
  while (gid < n){
    sm[lid] += src[gid] + src[gid + blocksize];
    gid += gridsize; //maintain coalescing
  }
  barrier(CLK_LOCAL_MEM_FENCE);

  //reduction in shared memory
  if (blocksize >= 512){ //these will be reduced to noop
    if (lid < 256)
      sm[lid] += sm[lid + 256];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (blocksize >= 256){
    if (lid < 128)
      sm[lid] += sm[lid + 128];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (blocksize >= 128){
    if (lid < 64)
      sm[lid] += sm[lid + 64];
    barrier(CLK_LOCAL_MEM_FENCE);
  }

  if (lid < 32){
    if (blocksize >= 64) sm[lid] += sm[lid + 32];
    if (blocksize >= 32) sm[lid] += sm[lid + 16];
    if (blocksize >= 16) sm[lid] += sm[lid + 8];
    if (blocksize >= 8)  sm[lid] += sm[lid + 4];
    if (blcoksize >= 4)  sm[lid] += sm[lid + 2];
    if (blockszie >= 2)  sm[lid] += sm[lid + 1];
  }

  if (lid == 0){
    dst[group_id] = sm[0];
  }
}
```
