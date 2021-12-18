# Ember
Semantic filesystem implemented in Rust.

Rather than files being stored by path, they are stored by ID with associated metadata.

For simplification of the implementation, the filesystem has the following limitations:
 - Each file must take up at least 64KiB of storage, or 65_536 bytes (same as WebAssembly page size)
 - Used pages must be stored in memory contiguously (wear leveling should be handled by hardware)
 - Files don't have to be stored contiguously (this will make the filesystem only work well on SSDs and Flash storage devices)

On a 500 GB hard drive this will give you a maximum of 8_388_608 files.

This means software must be designed to use less files than traditional software, by using zip files or similar.
An IDE should store a repository as a single file (including version control files).

-----

## Page 0
Page 0 is a special page that contains filesystem information.

 - 0: Magic Number For Ember Filesystem v1: `u128`
 - 16: Capacity (in pages: max ~ 1 yobibyte): `u64`
 - 24: Length (in pages: max = capacity): `u64`
 - 32: Size of file metadata list: `u64`
 - 40: Size of tag metadata list: `u64`
 - 48: Cached file ID that uses first non-metadata page `u56`
 - 55: Length of Drive Name: `u8`
 - 56: Drive Name: `[u8; 200]`
 - 256...: The rest of this page should be file metadata page (255 should fit).

-----

## Tag Metadata
A tag can have up to 6 files associated with it before it requires an index file.  Tags are 256 Bytes, they are listed in their own "file".

 - 0: Tag Type: `u8`
   - `0`: Name of file (example: `["My Song Title", "My First Song"]`)
   - `1`: Name of project(s) (example: `["My Album Name", "Music Composition"]`)
   - `2`: MIME Type of the file (example: `["audio/ogg", "audio/opus"]`)
 - 1: Tag Name Length: `u8`
 - 2: Files Length of Last Index Page (used for file index size when number of index pages is 0): `u16`
 - 4: Files Number of Index Pages: `u32`
 - 8: Files Index (up to 6 files) | Index Files (up to 40_960 files before index of index files required): `[u64; 6]`
 - 56: Tag Name Text: `[u8; 200]` (Limited to 50 characters)

-----

## File Metadata
File metadata is 256 bytes.  Also 256, and listed in their own file (like tag metadata).

 - 0: Creation Date: `u64`
 - 8: Modification Date: `u64`
 - 16: Last Access Date: `u64`
 - 24: Number of tags (up to 16, `0xFF` for a tag page): `u8`
 - 25: Number of file pages: `u8`
 - 26: Number of file page indices: `u16`
 - 28: Tag page: `u64`
 - 32: File Pages CRC-32: `[u32; 8]`
 - 64: File Pages Index (up to 8 pages) | Index Pages (up to 57344 pages, before index of page indices required): `[u64; 8]`
 - 128: Tags: `[u64; 16]`

-----

## File Page Index Page

 - 0: Page Indices For File: `[u64; 5461]`
 - 43688: CRC-32 For Pages For File: `[u32; 5461]`
 - 65532: CRC-32 For File Page Index Page: `u32`

-----
