#### 工作空间

打开 File -> Switch Workspace -> Other 菜单

Eclipse stores your projects in a folder called a workspace

![image-20220531151430424](https://img-note.langyastudio.com/202205311514481.png?x-oss-process=style/watermark)



#### 导入项目

打开 File -> Import 菜单

- 选择已存在的项目

![image-20220512102215872](https://img-note.langyastudio.com/202205121022913.png?x-oss-process=style/watermark)

- 选择项目的文件夹即可

![image-20220512102244895](https://img-note.langyastudio.com/202205121022935.png?x-oss-process=style/watermark)



#### 依赖包

选择项目，右键，从菜单中选择 Build path->Configure Build path

- 添加依赖库

![image-20220512101352155](https://img-note.langyastudio.com/202205121013199.png?x-oss-process=style/watermark)

- 可以通过 User Library 快速添加依赖库

  可以 Import 与 Export 进行依赖包配置的导入与导出处理

![image-20220512101509249](https://img-note.langyastudio.com/202205121015302.png?x-oss-process=style/watermark)



#### 运行配置

打开 Run -> Run Configurations 菜单

- Main 入口

  ![image-20220512095254519](https://img-note.langyastudio.com/202205120952584.png?x-oss-process=style/watermark)

- Arguments 参数

  ![image-20220512095352743](https://img-note.langyastudio.com/202205120953796.png?x-oss-process=style/watermark)

- JRE 运行虚拟机的版本![image-20220512095425299](https://img-note.langyastudio.com/202205120954352.png?x-oss-process=style/watermark)



#### 字符编码

![image-20220512095621233](https://img-note.langyastudio.com/202205120956283.png?x-oss-process=style/watermark)

![image-20220512110842574](https://img-note.langyastudio.com/202205121108622.png?x-oss-process=style/watermark)



#### Tomcat

打开 Window -> Preferences 菜单

- Tomcat Home 与 版本设置![image-20220512095923372](https://img-note.langyastudio.com/202205120959422.png?x-oss-process=style/watermark)

- JVM Settings 

  ![image-20220512100424399](https://img-note.langyastudio.com/202205121004451.png?x-oss-process=style/watermark)

  也可以编辑 `tomcat\bin\setclasspath.bat` 文件，在文件的开始添加如下设置

  ```bash
  set JAVA_HOME=C:\Java\jdk1.6.0_45
  set JRE_HOME=C:\Java\jre6
  ```



#### GUI 设计界面

[WindowBuilder](https://blog.csdn.net/stormdony/article/details/79497030)



#### 错误页面切换

Display Selected Console

![image-20220531185648737](https://img-note.langyastudio.com/202205311856817.png?x-oss-process=style/watermark)



#### JRE not compatible with project .class

选择项目，右键，从菜单中选择 properties->Java Compiler

- 设置允许项目特殊设置与jvm的版本

![image-20220512100804786](https://img-note.langyastudio.com/202205121008828.png?x-oss-process=style/watermark)



#### The import xxx cannot be resolved

- **clean** 项目，重新编译项目
  一般使用eclipse/myeclipse的菜单 project -> clean …可以解决。同时最好选中Build Automatically选项
- 若是没有解决，不要着急，继续来。**重新导入jar包**
  右键项目->build path -> Config build path -> Libraries-> remove后，重新导入
- 该项目有依赖项目需要重新导入依赖项目
  右键项目->build path -> Config build path -> project -> remove后，重新导入