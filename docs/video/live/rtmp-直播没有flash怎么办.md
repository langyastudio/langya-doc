各大主流浏览器在很早的时候就已声明 2020 年底不支持 Adobe Flash。所以已经线上运行的项目以及涉及直播的项目，都会涉及一个问题 ： “**没有 Adobe Flash 在 Web 浏览器端如何播放 RTMP 直播流**？”

> 还好有先见之明，我参与涉及直播的项目已经在 20 年初提前解决了该问题



#### 优选方案

需要流媒体服务器支持某种播放协议，例如HTTP-FLV、HLS等协议

- Web 浏览器

    HTTP-FLV、HLS

- 移动浏览器

    HLS、FLV（需要考虑兼容性）

- 移动Native or 小程序

    RTMP、HTTP-FLV、HLS

> HLS 延时高（5-10秒），可使用 [hls.js](https://github.com/video-dev/hls.js) 播放
>
> FLV 延时低（3-5秒），替代RTMP协议，可以使用 [flv.js](https://github.com/bilibili/flv.js) 播放，



#### Adobe 方案

https://www.flash.cn/

重庆重橙网络科技有限公司接管了 Flash，在中国区继续维护 Adobe —— 即**老版本**的浏览器可以通过更新 Flash 插件，继续使用 Flash

![image-20210120192533487](https://img-note.langyastudio.com/20210120192533.png?x-oss-process=style/watermark)



