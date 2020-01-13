## du命令和ls 显示文件的大小的区别

- `ls` shows the exact bytes of the files(显示文件open,read 的大小)
- `du` is showing the disk usage, that is different because the system store the files using blocks.(展示磁盘占用大小)
- `size` shows the size of the runtime image of an object/executable which is not directly related to the size of the file (bss uses no bytes in the file no matter how large, the file may contain debugging information that is not part of the runtime image, etc.) (展示动态运行时的大小)

