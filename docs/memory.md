---
title: Memory
---

# Architectural stack
Qiling abstracts the architectural stack operations as follows:

Pop a value off the top of stack and adjust stack pointer accordingly:
```python
value = ql.arch.stack_pop()
```

Push a value to the top of stack and adjust stack pointer accordingly:
```python
ql.arch.stack_push(value)
```

Peek the value at a certain offset from the top without modifying the stack pointer:
Note: the offset may be either positive, negative or zero (to peek the top of stack)
```python
value = ql.arch.stack_read(offset)
```

Replace a value at a certain offset from the top without modifying the stack pointer:
Note: the offset may be either positive, negative or zero (to replace the top of stack)
```python
ql.arch.stack_write(offset, value)
```

# Memory subsystem
Represents the emulated memory space. For more methods please refer to [qiling/os/memory.py](https://github.com/qilingframework/qiling/blob/master/qiling/os/memory.py)

## Managing memory
Qiling offers several methods for managing the emulated memory space:

| Method            | Description
|:--                | :--
| `map`             | Map a memory region at a certain location so it become available for access
| `unmap`           | Reclaim a mapped memory region
| `unmap_all`       | Reclaim all mapped memory regions
| `map_anywhere`    | Map a memory region in an unspecified location
| `protect`         | Modify access protection bits of a mapped region (rwx)
| `find_free_space` | Find an available memory region
| `is_available`    | Query whether a memory region is available
| `is_mapped`       | Query whether a memory region is mapped

Note: `is_available` and `is_mapped` are not necessarily ooposites; when a memory region is _partially taken_ (mapped), both methods will return `False`.


### Mapping memory pages
Memory has to be mapped before it can be accessed. The `map` method binds a contiguous memory region at a specified location, and sets its access protection bits. A string label may be provided for easy identification on the mapping info table (see: `get_map_info`).

Synposys:
```python
ql.mem.map(addr: int, size: int, perms: int = UC_PROT_ALL, info: Optional[str] = None) -> None
```
Arguments:
- `addr` - requested mapping base address; should be on a page granularity (see: `pagesize`)
- `size` - mapping size in bytes; must be a multiplication of page size
- `perms` - protection bitmap; defines whether this memory range is readable, writeable and / or executable (optional, see: `UC_PROT_*` constants)
- `info` - sets a string label to the mapped range for easy identification (optional)

Returns: `None`

Raises: `QlMemoryMappedError` if the requested memory range is not entirely available


### Unmapping memory pages
Mapped memory regions may be reclaimed by unmapping them. The `unmap` method reclaims a memory region at a specified location. The unmapping functionality is not limited to compelte memory regions, and may be used for partial ranges as well.

Synposys:
```python
ql.mem.unmap(addr: int, size: int) -> None
```
Arguments:
- `addr` - region base address to unmap
- `size` - region size in bytes

Returns: `None`

Raises: `QlMemoryMappedError` if the requested memory range is not entirely mapped


## Accessing Memory
The following methods may be used to access the emulated memory space:

| Method      | Description
|:--          | :--
| `read`      | Read a chunk of data from a specific location in memory
| `write`     | Write a chunk of data to a specific location in memory
| `read_ptr`  | A utility function to read an integer value of a certain size from memory
| `write_ptr` | A utility function to write an integer value of a certain size to memory

Note: only mapped ranges can be accessed. Accessing a non-mapped area will result in an exception.

### Reading data
Mapped memory may be accessed for reading in multiple ways as described below. Note that memory protections (that is, non-readable pages) do not apply to Qiling itself but only to memory accesses performed by the emulated program, therefore all mapped memory is readable by Qiling.

#### Read bytes from memory
Synposys:
```python
ql.mem.read(addr: int, size: int) -> bytearray
```
Arguments:
- `addr` - memory location to read data from; any address is valid as long as it is mapped
- `size` - number of bytes to read; any size is valid as long as the span area is mapped

Returns: a `bytearray` object containing the memory content

Raises: `UcError` if the requested memory range is not entirely available

#### Read an integer value from a memory address
Synposys:
```python
ql.mem.read_ptr(addr: int, size: int = 0, *, signed = False) -> int
```
Arguments:
- `addr` - memory address to read from
- `size` - pointer size (in bytes): either `1`, `2`, `4` or `8`; default is arch native size
- `signed` - interpret value as a signed integer (default: `False`)

Returns: integer value stored at the specified memory address

Raises: `UcError` if the requested memory range is not entirely available

### Writing data
Mapped memory may be accessed for writing in multiple ways as described below. Note that memory protections (that is, non-writeable pages) do not apply to Qiling itself but only to memory accesses performed by the emulated program, therefore all mapped memory is writeable by Qiling.

#### Write bytes to memory
Synposys:
```python
ql.mem.write(addr: int, data: bytes) -> None
```
Arguments:
- `addr` - memory location to write data to; any address is valid as long as it is mapped
- `data` - bytes to write; data in any size is valid as long as the span area is mapped

Returns: `None`

Raises: `UcError` if the requested memory range is not entirely available

#### Write an integer value to a memory address
Synposys:
```python
ql.mem.write_ptr(addr: int, value: int, size: int = 0, *, signed = False) -> None
```
Arguments:
- `addr`   - target memory address
- `value`  - integer value to write
- `size`   - pointer size (in bytes): either `1`, `2`, `4` or `8`; default is arch native size
- `signed` - interpret value as a signed integer (default: `False`)

Returns: `None`

Raises: `UcError` if the requested memory range is not entirely available


## Searching memory
Mapped memory may be searched for either a specific bytes sequence or a pattern.

Synposys:
```python
ql.mem.search(needle: Union[bytes, Pattern[bytes]], begin: Optional[int] = None, end: Optional[int] = None) -> List[int]
```
Arguments:
- `needle` - bytes sequence or regex pattern to look for
- `begin`  - search starting address; if not specified, start at lowest avaiable address
- `end`    - search ending address; if not specified, end at highest avaiable address

Returns: a list containing the addresses of all matches, or an empty list if there were no findings

Notes: memory ranges mapped as MMIO will be excluded due to potential side effects of the read operations

## Utilities

### Alignment
Some methods require the specified base address or size to be aligned on a certain boundary. The following utility methods help achieving that.

#### Align a value
Align a value down to the specified alignment boundary. If `value` is already aligned, the same value is returned.
Commonly used to determine the base address of the enclosing page.

Synposys:
```python
ql.mem.align(value: int, alignment: Optional[int] = None) -> int
```
Arguments:
- `value`     - a value to align
- `alignment` - alignment boundary; must be a power of 2. if not specified, value will be aligned to native page size

Returns: value aligned down to boundary

#### Align a value up
Align a value up to the specified alignment boundary. If `value` is already aligned, the same value is returned.
Commonly used to determine the end address of the enlosing page.

Synposys:
```python
ql.mem.align_up(value: int, alignment: Optional[int] = None) -> int
```
Arguments:
- `value`     - a value to align
- `alignment` - alignment boundary; must be a power of 2. if not specified, value will be aligned to native page size

Returns: value aligned up to boundary
