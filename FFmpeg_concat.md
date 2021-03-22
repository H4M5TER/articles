@[TOC](关于使用FFmpeg进行视频拼接)
# 方案
<http://trac.ffmpeg.org/wiki/Concatenate>
## 在音视频流级别拼接
[https://ffmpeg.org/ffmpeg-formats.html#concat](https://ffmpeg.org/ffmpeg-formats.html#concat)
> This demuxer reads a list of files and other directives from a text file and demuxes them one after the other, as if all their packets had been muxed together. 

从文本读取文件和指令列表以进行demux和mux
### 例子
    ffmpeg -f concat -i file.txt -c  copy output
```
examlple.txt

file 'video1'
duration 30
file 'video2'
inpoint 40
duration 20
```
## 在文件级别拼接
[https://ffmpeg.org/ffmpeg-protocols.html#concat](https://ffmpeg.org/ffmpeg-protocols.html#concat)
> Physical concatenation protocol.

ffmpeg支持强行物理拼接的协议(protocol)
> Certain files (MPEG-2 transport streams, possibly others) can be concatenated.

可以推测支持mpegts这类没有文件头部(startcode) 可以直接物理拼接的格式(多数为Transport Format)#
### 例子
    ffmpeg -i "concat:input1.ts|input2.ts|input3.ts" -c copy output.ts
### 更多支持
为了能够拼接mp4容器的视频 需要先转换为ts文件
#### 例子
    ffmpeg -i input1.mp4 -c copy -f mpegts intermediate1.ts
    ffmpeg -i input2.mp4 -c copy -f mpegts intermediate2.ts
    ffmpeg -i "concat:intermediate1.ts|intermediate2.ts" -c copy -f mp4 output.mp4
虽然略微复杂 但是不涉及编码改变 总归比重新压制要快多了
#### 管道
此处可以使用管道进行化简 在此贴上[此处](http://trac.ffmpeg.org/wiki/Concatenate#protocol)示例的Linux下命令 Windows下命令不多做研究

    mkfifo temp1 temp2
    ffmpeg -y -i input1.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts temp1 2> /dev/null & \
    ffmpeg -y -i input2.mp4 -c copy -bsf:v h264_mp4toannexb -f mpegts temp2 2> /dev/null & \
    ffmpeg -f mpegts -i "concat:temp1|temp2" -c copy -bsf:a aac_adtstoasc output.mp4

### 替换方案
> This is analogous to using cat on UNIX-like systems or copy on Windows.

这种协议和类unix系统中的cat和windows系统中的copy类似
此前网上也有见到[使用系统命令进行拼接 再使用FFmpeg输出的方案](https://blog.csdn.net/u011757360/article/details/20491055)
#### 例子
    copy /b "input1.ts" + "input2.ts" "output.ts"
    ffmpeg -i output.ts -c copy -f mp4 output.mp4
#### 管道
经过我思考 此处也可以使用管道化简如下 实测不可用

    copy /b "input1.ts" + "input2.ts" | ffmpeg -f mpegts -i - -c copy -f mp4 output.mp4
# 失败
## concat format
non monotonous dts in output stream
# 其他工具
## MP4Box
GPac附属工具
## MKVMerge
MKVToolNix附属工具
# 额外知识
## 管道
### Pipe
### Named Pipe
### Frameserver
## h264_mp4toannexb & aac_adtstoasc
转换到mpegts时需要h264_mp4toannexb
转换到mp4时需要aac_adtstoasc
现在目标为前者时都会自动添加bitstreamfilter后者
# 参考资料
[http://trac.ffmpeg.org/wiki/Concatenate](http://trac.ffmpeg.org/wiki/Concatenate)
[https://ffmpeg.org/documentation.html](https://ffmpeg.org/documentation.html)
[https://blog.csdn.net/flood_dragon/article/details/27539381](https://blog.csdn.net/flood_dragon/article/details/27539381)
# 修改历史
2019.09.16 08:10 又是写好几小时写一半太困了