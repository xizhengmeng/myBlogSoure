---
title: FLV格式解析
date: 2016-07-30 15:17:30
tags:
- 流媒体
categories: 基础

---
FLV 是 FLASH VIDEO 的简称,，FLV流媒体格式是一种新的视频格式，全称为Flash Video。FLV 压缩与转换非常方便，适合做短片.，并且 FLV 可以很好的保护原始地址，不容易下载到，可以起到保护版权的目的
<!--more-->


最近要用到flv，整理了一些flv格式的资料，供参考。 flv文件主要由两部分组成：header和body。

## 1.header

header部分记录了flv的类型、版本等信息，是flv的开头，一般都差不多，占9bytes。具体格式如下：

|文件类型	 | 3bytes | “FLV”|
|-----------|
|版本	| 1 byte|	一般为0x01|
|流信息	|1 byte	|倒数第一位是1表示有视频，倒数第三位是1表示有音频，倒数第二、四位必须为0|
|header长度|	4 bytes | 整个header的长度，一般为9；大于9表示下面还有扩展信息|

## 2.body

body部分由一个个Tag组成，每个Tag的下面有一块4bytes的空间，用来记录这个tag的长度，这个后置用于逆向读取处理，他们的关系如下图： flv
![](http://i1.piimg.com/567571/9a6b94bf91e209d3.jpg)
### 2.1.Tag
每个Tag由也是由两部分组成的：Tag Header和Tag Data。Tag Header里存放的是当前Tag的类型、数据区（Tag Data）长度等信息，具体如下：

|名称|	长度|	介绍|
|------|
|Tag类型 |1 bytes |8：音频 9：视频 18：脚本  其他：保留
|数据区长度	|3 bytes|	在数据区的长度|
|时间戳	|3 bytes	|整数，单位是毫秒。对于脚本型的tag总是0|
|时间戳扩展|	1 bytes	|将时间戳扩展为4bytes，代表高8位。很少用到
|StreamsID	|3 bytes	|总是0|

数据区(data)	由数据区长度决定	数据实体
### 2.2.Tag Data
数据区根据Tag类型的不同可分为三种，音频数据、视频数据和脚本数据。
#### 2.2.1.音频数据
第一个byte是音频的信息，格式如下。

|名称|长度	|介绍|
|------------|
|音频格式|	4 bits	|0 = Linear PCM, platform endian,1 = ADPCM,2 = MP3,3 = Linear PCM, little endian,4 = Nellymoser 16-kHz mono,5 = Nellymoser 8-kHz mono,6 = Nellymoser,7 = G.711 A-law logarithmic PCM,8 = G.711 mu-law logarithmic PCM,9 = reserved,10 = AAC,11 = Speex,14 = MP3 8-Khz,15 = Device-specific sound|
|采样率	|2 bits	|0 = 5.5-kHz,1 = 11-kHz,2 = 22-kHz,3 = 44-kHz,对于AAC总是3|
|采样的长度	|1 bit	|0 = snd8Bit,1 = snd16Bit,压缩过的音频都是16bit
|音频类型	|1 bit	|0 = sndMono,1 = sndStereo,对于AAC总是1|
第2byte开始就是音频流数据了。
#### 2.2.2.视频数据
和音频数据一样，第一个byte是视频信息，格式如下：

|名称	|长度	|介绍|
|----------------|
|帧类型	|4 bits	|1: keyframe (for AVC, a seekable frame)
2: inter frame (for AVC, a non-seekable frame)
3: disposable inter frame (H.263 only)
4: generated keyframe (reserved for server use only)
5: video info/command frame|
|编码ID	|4 bits	|1: JPEG (currently unused)
2: Sorenson H.263
3: Screen video
4: On2 VP6
5: On2 VP6 with alpha channel
6: Screen video version 2
7: AVC|
#### 2.2.3脚本数据
脚本Tag一般只有一个，是flv的第一个Tag，用于存放flv的信息，比如duration、audiodatarate、creator、width等。
首先介绍下脚本的数据类型。所有数据都是以数据类型+（数据长度）+数据的格式出现的，数据类型占1byte，数据长度看数据类型是否存在，后面才是数据。
其中数据类型的种类有：

- 0 = Number type
- 1 = Boolean type
- 2 = String type
- 3 = Object type
- 4 = MovieClip type
- 5 = Null type
- 6 = Undefined type
- 7 = Reference type
- 8 = ECMA array type
- 10 = Strict array type
- 11 = Date type
- 12 = Long string type

如果类型为String，后面的2bytes为字符串的长度（Long String是4bytes），再后面才是字符串数据；如果是Number类型，后面的8bytes为Double类型的数据；Boolean类型，后面1byte为Bool类型。

知道了这些后再来看看flv中的脚本，一般开头是0x02，表示String类型，后面的2bytes为字符串长度，一般是0x000a（“onMetaData”的长度），再后面就是字符串“onMetaData”。好像flv格式的文件都有onMetaData标记，在运行ActionScript的时候会用到它。后面跟的是0x08，表示ECMA Array类型，这个和Map比较相似，一个键跟着一个值。键都是String类型的，所以开头的0x02被省略了，直接跟着的是字符串的长度，然后是字符串，再是值的类型，也就是上面介绍的那些了。

## 3.总结

flv的格式还是比较简单的，header部分很简洁，body部分都是由一个个tag，tag的话也就三种，脚本tag一般只有一个的，我想这也是flv能成为在线视频格式的原因吧。只要了解了格式，我们就可以写个程序来解析flv文件了，这也是我下一步要做的。
最后附上flv官方手册：video_file_format_spec_v10
