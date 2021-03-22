# **关于使用FFmpeg进行视频拼接**

## **方案**

<http://trac.ffmpeg.org/wiki/Concatenate>

### **使用concat demuxer**

FFmpeg读取不同格式使用不同的demuxer，而concat也被视为一种"格式"。

<https://ffmpeg.org/ffmpeg-formats.html#concat>

> This demuxer reads a list of files and other directives from a text file and demuxes them one after the other, as if all their packets had been muxed together.  
> The timestamps in the files are adjusted so that the first file starts at 0 and each next file starts where the previous one finishes. Note that it is done globally and may cause gaps if all streams do not have exactly the same length.  
> All files must have the same streams (same codecs, same time base, etc.).  

个人理解，concat demuxer从文本文件读取文件和操作列表，然后将其视为一整个文件来进行读取。  
值得一提的是，concat demuxer能在stream级别工作，也就是说我们并非仅仅可以把两个文件拼在一起，甚至可以做到把视频1的前10秒和视频2的后20秒拼在一起，并把它视为一整个文件。

例子:

```text
file 'video1'
inpoint 0
outpoint 60
file 'video2'
inpoint 120
outpoint 180
```

```commandline
ffmpeg -f concat -i file.txt -c  copy output
```

### **使用concat协议**

FFmpeg支持很多输入和输出的协议，譬如file/http/rtmp，也包括concat。  

<https://ffmpeg.org/ffmpeg-protocols.html#concat>
> Physical concatenation protocol.

<https://trac.ffmpeg.org/wiki/Concatenate#protocol>
> This is analogous to using cat on UNIX-like systems or copy on Windows.

这是一个强行物理拼接的协议，就和Linux下cat命令/Windows下copy /b命令类似。  
自然的，仅支持MPEG-2 Transport Stream这类，没有文件头部(startcode)，可以直接物理拼接的格式。（用于传输的格式大多如此）

例子:

```commandline
ffmpeg -i "concat:input1.ts|input2.ts|input3.ts" -c copy output.ts
```

#### **更多支持**

为了能够拼接mp4容器的视频 需要先转换为ts文件  
例子:

```commandline
ffmpeg -i input1.mp4 -c copy -f mpegts intermediate1.ts
ffmpeg -i input2.mp4 -c copy -f mpegts intermediate2.ts
ffmpeg -i "concat:intermediate1.ts|intermediate2.ts" -c copy -f mp4 output.mp4
```

虽然略微复杂 但是不涉及编码改变 总归比重新压制要快多了

#### **其他方案**

上面提过，这种协议和类unix系统中的cat和windows系统中的copy类似，所以也可以不使用此协议，而使用系统命令替换。  
此前网上也有见到[使用系统命令进行拼接 再使用FFmpeg输出的方案](https://blog.csdn.net/u011757360/article/details/20491055)。  
例子:

```commandline
copy /b "input1.ts" + "input2.ts" "output.ts"
ffmpeg -i output.ts -c copy -f mp4 output.mp4
```

### **使用concat滤镜**

这种方法需要重编码，不过因此也支持拼接不同编码的视频，和预期目标不符，没有多做研究。  
等待研究FFmpeg的滤镜后补全。

## **失败**

在源两个视频裁切自同一个视频的情况下，使用concat demuxer拼接，显示 Non monotonous DTS in output stream错误，或许是time base问题，待深入研究。

## **其他工具**

### MP4Box

<https://gpac.wp.imt.fr/mp4box/> GPac附属工具

### MKVMerge

<https://mkvtoolnix.download/> MKVToolNix附属工具

### MEncoder

<http://www.mplayerhq.hu/> MPlayer附属工具

## **额外知识**

### **管道**

\*nix和Windows的命令行下都有把一个程序输入链接到另一个程序输出的管道功能。  
\*nix的命令行下还有命名管道(FIFO文件类型)可以作为两个程序的中介，avisynth和vapoursynth这种所谓frameserver应该就可以说是一种命名管道。

### **h264_mp4toannexb & aac_adtstoasc**

在查阅的资料中常常看到这两个bitstream filter参数。  
<http://ffmpeg.org/ffmpeg-bitstream-filters.html>  
查阅得转换到mpegts时需要h264_mp4toannexb参数，转换到mp4时需要aac_adtstoasc参数。在转换时FFmpeg会自动添加，不需要手动填写。  
具体原理待深入研究。

## 参考资料

<https://blog.csdn.net/doublefi123/article/details/47276739>  
<http://trac.ffmpeg.org/wiki/Concatenate>  
<https://ffmpeg.org/documentation.html>  
<https://yonsm.ga/mp4merge/>  
<https://blog.csdn.net/u011757360/article/details/20491055>  
<http://itindex.net/detail/52379-ffmpeg-%E5%90%88%E5%B9%B6-%E8%A7%86%E9%A2%91>

## 修改历史

2019/09/16 08:10 又是熬夜到天亮才写一半 太困了  
2019/09/23 12:08 正确的formatted了 修改补充了一些内容  
