### 对比

> https://github.com/only-cliches/NoProto
>
> flatbuffer 就是内存直接映射，所以最快
> protobuffer 是二进制，做了一点点压缩，所以最小，但是速度稍慢
> json 是文本化的，又大又慢，但是可读性强

- 编解码由快到慢

  flatbuffer > protobuffer > json

- size 由小到大

  protobuffer < flatbuffer < json



### FlatBuffer

http://google.github.io/flatbuffers/flatbuffers_guide_use_java.html



### Fury

![image-20221206141006159](https://img-note.langyastudio.com/202212061410228.png?x-oss-process=style/watermark)

![image-20221206140955735](https://img-note.langyastudio.com/202212061409822.png?x-oss-process=style/watermark)