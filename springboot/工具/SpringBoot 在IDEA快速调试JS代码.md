# SpringBoot 在IDEA快速调试JS代码

> 让JS的Debug模式变得跟Java Debug一样简单，是时候对Chrome的F12的控制台窗口说拜拜了！

# 学习目标

简单三步！快速学会IDEA JS Debug调试。

# 快速查阅

官方教程： [Debugging JavaScript in Chrome](http://www.jetbrains.com/help/idea/debugging-javascript-in-chrome.html)

# 使用教程

- 安装浏览器插件（JetBrains IDE Support ）

打开谷歌浏览器的应用商店，如被墙请先使用[谷歌访问助手](http://www.ggfwzs.com/)，然后搜索并安装JetBrains IDE Support 这款由IDEA官方提供的插件。

![img](https:////upload-images.jianshu.io/upload_images/8069210-61c1acb74339b1a3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



- 添加启动入口

在IDEA的菜单栏打开Edit Configurations ，点绿色“+”号选择JavcScript Debug ，然后填写需要进行JS调试的地址URL，底下的Project会自动生成。URL 项目访问根URL

![img](https:////upload-images.jianshu.io/upload_images/8069210-9e794aacb005e904.png?imageMogr2/auto-orient/strip|imageView2/2/w/1109/format/webp)

- 启动DEBUG调试

在你需要的JS打上断点，启动这个JS DEBUG项目，然后进入断点回自动返回IDEA进行调试，如图。

![img](https:////upload-images.jianshu.io/upload_images/8069210-7f29400fde70316e.png?imageMogr2/auto-orient/strip|imageView2/2/w/877/format/webp)

