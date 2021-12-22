发现问题在于新版本的 ImageMagick 启用了 **OpenMP**，默认开启多线程处理，导致 cpu 占用过高。**convert** 占用 cpu 很高。

首先用 `/usr/local/imagemagick/bin/convert -version` 指令查看一下输出內容是否已经开启了多线程

- Features 的值为空或DPC，说明是单线程
- Features的值是 openMP，说明是多线程

> imagick 的多线程模式有一個 bug，他会导致多核心的 cpu 使用率瞬间飙升到 100%。所以一定要使用它的单线程程模式才行。



**解决办法：**
重新编译 ImageMagick 最新版，增加了参数： **--disable-openmp**，负载就降下来了

```bash
./configure --enable-shared --enable-threads=posix --with-system-zlib --enable-__cxa_atexit --disable-openmp
make && make install
```

> 如果您刚开始是用多线程模式安裝的imagick，那就必须要 `yum remove imagemagick` 將其卸载掉重新安裝才行