# XZ vs GZ

While .gz is stil the most used compression format, .xz is a solid 
alternative when compression ratio matters.

## Let's compare GZ and XF

This question have been discussed all around the web for years. Here are 
some links:

* https://unix.stackexchange.com/questions/108100/why-are-tar-archive-formats-switching-to-xz-compression-to-replace-bzip2-and-wha
* https://stackoverflow.com/questions/6493270/why-is-tar-gz-still-much-more-common-than-tar-xz
* https://lists.fedoraproject.org/pipermail/infrastructure/2010-August/009389.html

To sum-up:

* gz if faster, needs lets memory/CPU, but has a lower compression 
ratio
* xz benefits of the LZMA/LZMA2 compression algorithm - so has better 
compression ratio, but is slower at compression/decompression time and 
need more memory/CPU (and is less spread).

So GZ is the choice for inter-operability or embedded systems. XZ is the 
choice when file transfer time matter, and source/destination systems 
are recent and powerful.

## Usage

### GZ
Compression:
```
gzip <filename>
tar -czf <archive>.tar.gz <filename>
```

Uncompression:
```
gunzip <filename>.gz
tar -xzf <archive>.tar.gz
```

### XZ
Compression:
```
xz <filename>
tar -cJf <archive>.tar.xz <filename>
```
Compression ratio can be changed `xz -k9 <filename>`. Default is 6.

Uncompression:
```
xz -d <filename>.xz
tar -xJf <archive>.tar.xz
```

