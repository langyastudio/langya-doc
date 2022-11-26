> https://mp.weixin.qq.com/s/PBBnw0fr_ww5G32RY5VulA

## **什么是 MinIO**

Minio 是个基于 Golang 编写的开源对象存储套件，基于 Apache License v2.0 开源协议，虽然轻量，却拥有着不错的性能。它兼容亚马逊 S3 云存储服务接口。可以很简单的和其他应用结合使用，例如 NodeJS、Redis、MySQL 等。

### 应用场景

MinIO 的应用场景除了可以作为私有云的对象存储服务来使用，也可以作为云对象存储的网关层，无缝对接 `Amazon S3` 或者 `MicroSoft Azure` 。

![图片](https://img-note.langyastudio.com/202208242220805.jpeg?x-oss-process=style/watermark)



### 特点

- **高性能**：作为一款高性能存储，在标准硬件条件下，其读写速率分别可以达到 `55Gb/s` 和 `35Gb/s`。并且 MinIO 支持一个对象文件可以是任意大小，从几 kb 到最大 5T 不等

- **可扩展**：不同 MinIO 集群可以组成联邦，并形成一个全局的命名空间，并且支持跨越多个数据中心

- **云原生**：容器化、基于 K8S 的编排、多租户支持

- **Amazon S3 兼容**：使用 Amazon S3 v2 / v4 API。可以使用 Minio SDK，Minio Client，AWS SDK 和 AWS CLI 访问Minio 服务器

- **SDK支持**：

  - GO SDK：https://github.com/minio/minio-go

  - JavaSDK：https://github.com/minio/minio-java

  - PythonSDK：https://github.com/minio/minio-py

- **图形化界面**：有操作页面
- **支持纠删码**：MinIO 使用纠删码、Checksum 来防止硬件错误和静默数据污染。在最高冗余度配置下，即使丢失 1/2的磁盘也能恢复数据
