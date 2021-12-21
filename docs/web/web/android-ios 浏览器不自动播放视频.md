### 浏览器支持

相信很多 Web 前端开发小伙伴因为工作的需求，在研究怎么通过 HTML5 实现视频在手机浏览器的自动播放（主流浏览器）。在这里，我要告诉大家：

  1. Chrome for Android 从版本54开始支持**静音**视频自动播放
  2. Safari for iOS 10 从版本602开始支持**静音**视频自动播放
  3. Autoplay, whether muted or not, is already supported on Android by Firefox and UC Browser: they do not block any kind of autoplay



### 主要原因

为了通过视频解决大体积的 GIF 动画问题。由于动画 GIF 可能会变得非常大，视频通常要小得多。这就是 Apple 和 Google 决定在其移动浏览器中允许自动播放的原因。但前提是视频静音。在 iOS 上，还必须设置 playsinline 属性，因为默认情况下视频将全屏播放。

| 系统/浏览器                      | 是否支持自动播放                          |
| -------------------------------- | ----------------------------------------- |
| iOS 9 Safari 601                 | no                                        |
| iOS 10 Safari 602                | yes (with muted + playsinline attributes) |
| iOS 9 Chrome 54                  | no                                        |
| iOS 10 Chrome 54                 | yes (with muted + playsinline attributes) |
| iOS 9 Opera Mini 14              | no                                        |
| iOS 10 Opera Mini 14             | yes (plays fullscreen)                    |
| iOS 9 Firefox 5.3                | no                                        |
| iOS 10 Firefox 5.3               | no                                        |
| Android Chrome 43                | no                                        |
| Android Chrome 54                | yes (with muted attribute)                |
| Android Opera Mini 20            | no                                        |
| Android Firefox 49               | yes                                       |
| Android Samsung Internet 4.0     | no                                        |
| Windows Phone 8 IEMobile 10      | no                                        |
| Windows Phone 8.1 IEMobile 11    | yes                                       |
| Windows 10 Mobile Edge 14.1      | no                                        |
| Windows 10 Mobile Opera Mini 9.1 | no                                        |



参考：
[http://walterebert.com/blog/html5-video-autoplay-mobile-revisited/](http://walterebert.com/blog/html5-video-autoplay-mobile-revisited/)