# lemons asset archive (LEMNARC) specification

LEMNARC is a file format for storing multiple asset files into one archive

# LEMNARCv1

## quirks

### compression
compression is handled by zlib, using the "best speed" option

### encryption
archive encryption is provided by 2048-bit rsa, and file encryption is provided by 256-bit aes.

the encryption key for the archives and files is unique for every build of the archive, unless a static key pair is specified.

## file structure

### header

here's an example header of a LEMNARCv1 file:

```
Hex View  00 01 02 03 04 05 06 07  08 09 0A 0B 0C 0D 0E 0F
 
00000000  4C 45 4D 4E 41 52 43 00  01 00 00 00 00 00 00 00  LEMNARC.........
00000010  45 58 41 4D 50 4C 41 52  43 48 49 56 45 30 30 30  EXAMPLARCHIVE000
00000020  32 30 32 35 2D 30 35 2D  32 39 54 31 31 3A 34 32  2025-05-29T11:42
00000030  2B 30 30 3A 30 30 00 00  2A 00 00 00 00 00 00 00  +00:00..*.......
```

| example file                                     							| name                   | bytes | offset |
| ------------------------------------------------------------------------- | ---------------------- | ----- | ------ |
| `4C 45 4D 4E 41 52 43 00`                          						| magic bytes (LEMNARC)  | 0x08  | 0x00   |
| `01`                                               						| version                | 0x01  | 0x08   |
| `00 00 00 00`                                        						| attributes             | 0x04  | 0x0C   |
| `45 58 41 4D 50 4C 41 52 43 48 49 56 45 30 30 30`  						| archive identifier     | 0x10  | 0x10   |
| `32 30 32 35 2D 30 35 2D 32 39 54 31 31 3A 34 32 2B 30 30 3A 30 30 00`    | build date 			 | 0x17  | 0x20   |
| `2A 00 00` 																| file count 			 | 0x03	 | 0x38   | 

#### magic bytes
its just the string 'LEMNARC' with a null byte terminating it

#### version
this defines the version of LEMNARC that's used in this archive. for LEMNARCv1 it is `01`

#### attributes
it's 32 bits of archive flags

the flags are shown below
```
	E0000000 00000000 00000000 00000000
	
	E - Encrypted archive

```
#### archive identifier
a 16-byte long string that holds the name of the archive

#### build date
the date that this archive was compiled, stored in ISO 8601 format, terminated in a null byte

#### file count
a 24-bit unsigned integer that stores the ammount of files that this archive contains

after this header, at offset 0x40, there could be two things that might be placed. if the "encrypted" archive attribute was set, the following will be an encrypted version of the TOC and data. if not then the regular TOC and data follow

### table of contents (TOC)

the TOC is a list of the metadata about the files contained in the archive

every entry is variable length, but are aligned to every 0x10 hex

here's an example entry for the TOC

```
Hex View  00 01 02 03 04 05 06 07  08 09 0A 0B 0C 0D 0E 0F
 
00000000  AA BB CC DD 00 00 00 00  00 00 00 00 00 00 01 40  ...............@
00000010  00 00 00 00 00 00 02 5E  2F 68 6F 6D 65 2F 70 6C  .......^/home/pl
00000020  69 6E 6B 2F 65 78 61 6D  70 6C 65 2F 61 73 73 65  ink/example/asse
00000030  74 73 2F 68 65 6C 6C 6F  2E 77 61 76 00 00 00 00  ts/hello.wav....
```

| Example File                                     							| Name                    			 | Bytes | Offset |
| ------------------------------------------------------------------------- | ---------------------------------- | ----- | ------ |
| `AA BB CC DD`                				          						| file identifier (hash)  			 | 0x04  | 0x00   |
| `00 00`                				          							| file attributes		  			 | 0x02  | 0x04   |
| `00 00 00 00 00 00 01 40`                				   					| file offset						 | 0x08  | 0x08   |
| `00 00 00 00 00 00 02 5E`                									| file size							 | 0x08  | 0x10   |
| `2F 68 6F 6D 65 2F 70 6C 69 6E 6B 2F 65 78 61 6D 70 6C 65 2F 61 73 73 65 74 73 2F 68 65 6C 6C 6F 2E 77 61 76`  | file path | variable  | 0x18   |

#### file identifier
this is a 32-bit hash of the file that this entry belongs to (TODO: find a hash alg for this)

#### file attributes
it's 16 bits of file flags
```
	ZOE00000 00000000

	Z - zlib compressed
	O - file path omitted
	E - encrypted file
```

#### file offset
its the offset of where the file is located in 64 bits

#### file size
64 bits representing the size of the file

#### file path
if the 'file path omitted' flag isn't set, this contains a string of where the file is placed in the filesystem of the machine this archive was built on

### files
this section is a complete battlefield, there are no rules. other than the files have to be aligned to every 4096 bytes.
