# ExtendedMemory
[[عربي]](readme.ar.md)

Extension for the `Srl.Memory` module.

## Adding to the Project

We can install this library using the following statements:

```
import "Apm";
Apm.importFile("Alusus/ExtendedMemory");
```

## Example

```
import "Srl/Console";
import "Srl/Memory";
import "Apm";
Apm.importFile("Alusus/ExtendedMemory");
use Srl;

// Execute a program with pre-allocated memory block to avoid the overhead of repetitive
// memory allocations.
Memory.runWithPreallocation(4 * 1024 * 1024 /* 4 MB */, 4 * 1024 * 1024, false, closure() {
    // Program body here.
    ...
});
```

## Functions

### startPreallocation

```
func startPreallocation(size: ArchInt, enlargementSize: ArchInt): ptr;
```

This function preallocates a block of memory to be used in following memory allocation operations
to improve performance. After calling this function all calls to `Memory.alloc` will get blocks
within this preallocated memory block, and all calls to `Memory.free` will simply be ignored.
Instead, all allocated memory will eventually be freed in one shot during the following call
to `endPreallocation`.

Pay attention to the fact that all allocations happening after calling this function will
automatically be freed when `endPreallocation` is called, even if `Memory.free` was not
called on those blocks. This is what makes this improve performance, as it reduces the number
of actually memory allocation and deallocation. This means that the user must be cautious
not to allocate any memory block that is intended to be used after `endPreallocation` is
called.

Each call to this function must be accompanied by a follow up call to `endPreallocation`.
This function can also be called in nested manner, i.e. the user can call `startPreallocation`
then afterwards make another call to `startPreallocation`, and then call `endPreallocation`
twice.

Arguments:
* size: The size of the memory block that will be preallocated during the call to this function.
* enlargementSize: The size of extra memory blocks that will be allocated during calls to
  `Memory.alloc` whenever the existing blocks run out of empty space.

Return Value: A pointer to eventually be passed to `endPreallocation`.

### endPreallocation

```
func endPreallocation(block: ptr);
func endPreallocation(block: ptr, printLog: Bool);
```

Ends the preallocation of the specified call to `startPreallocation` and frees all memory allocated since
that call.

Arguments:
* block: The pointer recieved from `startPreallocation`.
* printLogs: Setting this to true enables debug logs that prints info about allocated blogs. This info
  is useful in verifying whether your code is leaking any memory blocks.

### runWithPreallocation

```
func runWithPreallocation(
    size: ArchInt, enlargementSize: ArchInt, printLog: Bool, toRun: closure()
);
```

This is just a helper function that executes the provided closure in preallocation mode. Refer to
`startPreallocation` and `endPreallocation` for info about the arguments.

