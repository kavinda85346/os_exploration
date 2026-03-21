### Archiving and Compresion

##### Given below are default commands in Linux which are used for view archiving and compresion.
1. Extract and archive a file.
```
#Extract and archive a file.
tar -xvf archive.tar
```
2. Archiving a file_1.
```
#Archiving a file_1
find . -name "*.txt" | cpio -ov > file_1.cpio
```
3. Archving tools for libraries.
```
ar rcs libexample.a file1.o file2.o
```
4. Compress a file.
```
#Compress a file.
gzip file.txt
bzip2 file.txt
xz file.txt    #high compression ratio
7z a archive.7z file1 file2    #7z compress
```

5. Decompress a file.
```
#Decompress a file.
gunzip file.txt.gz
bunzip2 file.txt.bz2
unxz file.txt.xz
7z x archive.7z
```
