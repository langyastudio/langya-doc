`Thumbnailator` 是一个优秀的图片处理的 Google 开源 Java 类库，专门用来生成图像缩略图，通过很简单的 API 调用即可生成图片缩略图，也可直接对一整个目录的图片生成缩略图。两三行代码就能够从现有图片生成处理后的图片，且允许微调图片的生成方式，同时保持了需要写入的最低限度的代码量。可毫不夸张的说，它是一个处理图片十分棒的开源框架。



## 简介

### Why

Making high-quality thumbnails in Java can be a fairly difficult task.

Learning how to use the Image I/O API, Java 2D API, image processing, image scaling techniques, ... but fear not! *Thumbnailator* will take care of all those things for you!

Thumbnailator is a single JAR file with no dependencies to external libraries, making development and deployment simple and easy. It is also available on the Maven Central Repository for easy inclusion in Maven projects.



官网：

https://code.google.com/p/thumbnailator/



### 支持

- 图片缩放
- 区域裁剪
- 水印
- 旋转
- 保持比例



## 使用

引入：

```xml
<dependency>
  <groupId>net.coobird</groupId>
  <artifactId>thumbnailator</artifactId>
  <version>0.4.14</version>
</dependency>
```



说明：

- `keepAspectRatio(boolean arg0)` 

  图片是否按比例缩放（宽高比保持不变）默认 `true`

- `outputQuality(float arg0)` 

  图片质量

- `outputFormat(String arg0)` 

  格式转换




样例：

```java
/**
 * 指定大小缩放 若图片横比width小，高比height小，放大 
 * 若图片横比width小，高比height大，高缩小到height，图片比例不变
 * 若图片横比width大，高比height小，横缩小到width，图片比例不变 
 * 若图片横比width大，高比height大，图片按比例缩小，横为width或高为height
 * 
 * @param resource  源文件路径
 * @param width     宽
 * @param height    高
 * @param tofile    生成文件路径
 */
public static void changeSize(String resource, int width, int height, String tofile) {
	try {
		Thumbnails.of(resource).size(width, height).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}

/**
 * 指定比例缩放 scale(),参数小于1,缩小;大于1,放大
 * 
 * @param resource   源文件路径
 * @param scale      指定比例
 * @param tofile     生成文件路径
 */
public static void changeScale(String resource, double scale, String tofile) {
	try {
		Thumbnails.of(resource).scale(scale).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}

/**
 * 添加水印 watermark(位置,水印,透明度)
 * 
 * @param resource  源文件路径
 * @param p         水印位置
 * @param shuiyin   水印文件路径
 * @param opacity   水印透明度
 * @param tofile    生成文件路径
 */
public static void watermark(String resource, Positions p, String shuiyin, float opacity, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).watermark(p, ImageIO.read(new File(shuiyin)), opacity).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}

/**
 * 图片旋转 rotate(度数),顺时针旋转
 * 
 * @param resource  源文件路径
 * @param rotate    旋转度数
 * @param tofile    生成文件路径
 */
public static void rotate(String resource, double rotate, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).rotate(rotate).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}

/**
 * 图片裁剪 sourceRegion()有多种构造方法，示例使用的是sourceRegion(裁剪位置,宽,高)
 * 
 * @param resource  源文件路径
 * @param p         裁剪位置
 * @param width     裁剪区域宽
 * @param height    裁剪区域高
 * @param tofile    生成文件路径
 */
public static void region(String resource, Positions p, int width, int height, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).sourceRegion(p, width, height).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

