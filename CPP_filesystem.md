### C++17 Filesystem header
C++17 introduces filesystem header for easy loading up attributes of a file, such as its size.

you can use the simple version for file size access:
```
try {
  std::filesystem::file_size("name");
} catch (fs::filesystem_error& e){
}
```

or have it report error code

```
std::error_code ec;
size_t size = std::filesystem::file_size("name", ec);
```
