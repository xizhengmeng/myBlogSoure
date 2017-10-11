---
title: Mac中的二进制文件
date: 2017-10-09 18:31:22
tags:
- iOS进阶
---
在Unix系统中，任何文件都可以通过chmod+x命令来标记为可执行文件，但是这并不能保证这个文件是能够被执行的，这个标志只是告诉系统这个文件可以读入内容，然后寻找一个头签名，据此可以确定精确的可执行文件格式，这个文件签名通常称为"magic"。
例如:

- #!脚本
- 0xcafebabe/0xbebafeca包含多种架构支持的二进制，又被称为通用二进制或者胖二进制
- 0xfeedface(32位)，0xfeedfacf(64位)Mach-O格式或者叫OSX原生二进制格式

### 胖二进制
通用二进制只不过是其支持的各种架构的二进制文件的打包文件，这种格式的文件包含一个文件头，然后文件头后面依次拷贝了每一种支持架构的二进制文件。
- 通用二进制文件的处理工具叫做`lipo`脂肪的意思，对应胖二进制的意思
- 这个工具可以提取，删除和替换通用二进制文件中制定架构的二进制代码，因此可以通过这个工具对二进制文件进行瘦身

### Mach-O

首先看mach_header
`otool -hV JDMobile`
```
Mach header

magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags

MH_MAGIC_64   ARM64        ALL  0x00     EXECUTE    68       7464   NOUNDEFS DYLDLINK TWOLEVEL WEAK_DEFINES BINDS_TO_WEAK PIE
```

- magic魔数代表32或者64位
- cputype和subcuptype代表cup类型
- filetype代表文件类型(可执行文件，库文件，核心转储文件，内核扩展)
- ncmds和sizeofcmds用于加载的家在命令的条数和大小
- flags代表动态连接器(dyld)的标志



