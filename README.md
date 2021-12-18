# ember
Semantic filesystem implemented in Rust

----

Rather than files being stored by path, they are stored by ID with associated metadata.

For simplification of the implementation, the filesystem has the following limitations:
 - Each file must take up at least 64KiB of storage, or 65_536 bytes (same as WebAssembly page size)
 - Used pages must be stored in memory contiguously (wear leveling should be handled by hardware)
 - Files don't have to be stored contiguously (this will make the filesystem only work well on SSDs and Flash storage devices)

On a 500 GB hard drive this will give you a maximum of 8_388_608 files.

This means software must be designed to use less files than traditional software, by using zip files or similar.
An IDE should store a repository as a single file (including version control files).

----

## File 0

File 0 contains basic information about the filesystem.

```rust
struct File0 {
    /// 0: Number of pages possible (maxes out around a yobibyte of data)
    capacity: u64,
    /// 8: Number of pages used by the filesystem
    length: u64,
    /// 16: Number of pages used by file 0
    pages: u64,
    /// 24: Number of files
    files: u64,
    /// 32: Magic Number
    magic: u128 = ????,
    /// 48: Reserved
    reserved: u64,
    /// 56: 200-bytes / 50 character enforced limit UTF-8 Name
    drive_name: [u8; 200],
    /// 256: 
}
```
