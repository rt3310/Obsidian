## Shared Libraries

### Static libraries have the following disadvantages
- Potential for duplicating lots of common code in the executable files on a file system
	- Every C program needs the standard C library
- Potential for duplicating lots of code in the text segment of each process
- Minor bug fixes of system libraries require cach application to explicitly relink
### Solution: Shared libraries
- Whose members are dynamically loaded and linked at runtime
- Called shared objects (.so) in UNIX or dynamic link libraries (DLL) in Windows
- Shared library routines can be shared by multiple processes
	- There is exactly one .so file for a particular library in any given file system
		- The code and data in this .so file
	- A single copy of the .text section of a shared library in memory can be shared by multiple processes
- Dynamic linking can occur when executable is first loaded and run
	- Common case for Linux. handled automatically by ld-linux.so
- Dynamic linking can also occur after program has begun
	- An application can request the dynamic linker to load and link shared library
	- In Linux, this is done explicitly by user with `dlopen()`