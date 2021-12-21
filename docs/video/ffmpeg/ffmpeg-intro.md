## 拼接

### 目录中的全部文件

Shell 按自然数排序获取一行一个文件的文件列表，并在每行前面加 **file空格**，匹配 ffmpeg 拼接清单格式，保存为文本文件

> ls -1v|sort -V|sed 's/^/file\ /' > manifest.txt

利用 ffmpeg 进行拼接，`-map 0 -r 25 -c copy -bsf:a aac_adtstoasc`

> ffmpeg -f concat -i manifest.txt output.mp4

