> 本文来自佚名，郎涯进行简单排版与补充

## Portainer

![图片](https://img-note.langyastudio.com/202212080913746.png?x-oss-process=style/watermark)

Portainer 是一个轻量级的 WEB 管理 UI ，可让你轻松管理运行在 Docker、Swarm、Kubernetes 环境下的容器。Portainer 提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm 集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。全面支持 Linux、Mac OS、Windows 主流操作系统。

![图片](https://img-note.langyastudio.com/202212080913918.jpeg?x-oss-process=style/watermark)

目前使用它来监控管理 Docker 容器，感觉它还是很强大的，非常好用。不足之处就是远程终端非常慢，不方便有时候远程进入容器内部进行操作。不过这不是拒绝它的理由，大部分日常都可以通过它很好的解决。

官方提供了**Portainer 演示项目**（账密：*admin* / *tryportainer*），如果有兴趣可以亲自感受一下。

> http://demo.portainer.io/



## DockStation

DockStation 是另一款 Docker 管理图形化界面，它比 Portainer 好的地方在于在多项目管理上非常清晰。

![图片](https://img-note.langyastudio.com/202212080914742.png?x-oss-process=style/watermark)

尤其能够图形化展示容器之间的依赖关系，尤其擅长管理 Docker-Compose，额外的它还支持监控统计、端口监视。而且 UI 设计的非常漂亮、非常清新，如果你希望对容器进行层次分明的管理的话不妨试一试它，它也支持 Linux、Mac OS、Windows 主流操作系统。最大的问题在于维护并不是特别活跃，不过不影响日常使用。



## Docker Dashboard

这是 Docker 官方的 Docker Desktop 提供的功能，亲儿子级别，功能比较单一，只提供了容器镜像的简单管理，容器的简单监控统计。

![图片](https://img-note.langyastudio.com/202212080914455.png?x-oss-process=style/watermark)

优点就是官方提供，缺点就是功能比较简单，只能管理本地的容器和镜像，另外目前只支持 Mac OS 和 Windows。也就是说只符合日常开发用用。



## 其他

Lazydocker 和 Docui 也是比较轻量的管理工具，只不过它们不算用户图形界面，只是强化版的终端。如果你有兴趣可以玩一玩。

这是 Lazydocker：

![图片](https://img-note.langyastudio.com/202212080914184.png?x-oss-process=style/watermark)Lazydocker

这是 Docui：

![图片](https://img-note.langyastudio.com/202212080914040.png?x-oss-process=style/watermark)Docui



## 总结

如果您需要团队级别的图形化管理工具，配合 Docker swarm，Docker，K8S 一起使用并且可以部署在远程服务器上，请选择 Portainer。如果您需要管理多项目，喜欢比较清新的 UI 也可以选择 DockStation。本地开发就用官方的 Dashboard 就可以了。而 Lazydocker 和 Docui 适合比较极客的开发者。