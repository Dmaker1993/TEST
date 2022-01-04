---
title: 基于Markdown语法的个人简历
top: false
date: 2021-12-29 12:15:38
tags:
    - 经验
    - 个人简历
    - markdown
categories:
    - 经验分享
---
# 基于Markdown语法的个人简历

作为程序员，markdown是我写文档最经常用的一种方式。刚好简历该更新了，所以想到了使用markdown写一份简历，简洁方便。但是因为是第一次，加上css也不是特别熟悉，写起来也是耗费了些时间的。

## 1、简历格式

在我百度了几次之后，发现使用markdown写简历最常用的格式就是竖着排的，就像下面这样（使用的[冷熊简历](http://cv.ftqq.com/?fr=github)模板）：
![在这里插入图片描述](https://img-blog.csdnimg.cn/89db22bcc51449a09bb59f257b7a56b5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


但是我觉得差点意思，我更喜欢类似表格的方式（[全民简历](https://www.qmjianli.com/cv/details/1004)模板），例如：

![在这里插入图片描述](https://img-blog.csdnimg.cn/ec3247277830460b9ba61c73546f905a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

## 2、完善简历

所以自己搞起来！首先要选一个主题，经过多方对比，最红选择了 `typora`自带的 `Whitey	` 。接下来就是完善内容了，我主要写了 *个人信息*、*工作经历* 、*专业技能* 和 *项目经历* 四大块内容。   开始我直接用空格把列之间隔开，在 `typora` 中展示没啥问题，很完美，但是当导出为 `pdf` 的时候，悲剧了，信息没对齐，丑爆了！！！

![在这里插入图片描述](https://img-blog.csdnimg.cn/ddb7990ccd0c46b0b40ab365450c00b3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

如果直接写 `css样式` ，可能会比较麻烦，所以使用 `表格` 来让信息对齐。但是问题如下：

1）表格的第一行是默认加粗的 ，跟其他内容格格不入
2）表格宽度会随着内容的长短而变化，导致多个表给之间的宽度不同

![在这里插入图片描述](https://img-blog.csdnimg.cn/18244c3946fa49d2814f1bae43962fe8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


这两个问题我是通过 `css样式` 来解决。但是在 `typora`中是不会能看到样式的，只有导出来之后才能看到效果，但是倒出来的 `pdf` 我也不知道该把样式加在哪些标签或者类上，突然想到先导出一个 ` html` ，那样就能直接调整html的样式，然后直接把样式复制到 md 文件中了。导出 `html` 后，在浏览器中打开，这就到了程序员的地盘了，样式随我们改。一通操作之后，我觉得效果还行，比较简洁：

![在这里插入图片描述](https://img-blog.csdnimg.cn/136f5335cce54a09bc7dd90fb7c4decc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARE1ha2VyMTk5Mw==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)


## 3、总结

使用markdown写个人简历是我比较喜欢的方式，因为它比较简洁，而且样式我们可以自己调整，高度定制化。我总结了几个需要注意的点：

1）选一个好看的主题。比如 `Whitey`。
2）当一行有多列的时候，一定要使用 `表格` 。因为不用表格的话，列的样式会乱。
3）如果样式看着比较别扭或者不合适，可以适当使用 `css` 样式调整。
4）导出看效果，调整到满意为止。

如果你也喜欢我这种样式的话，可以到github或者gitee去下载，包含了样式的。我是用md编辑器是 `typora`,	使用的主题是 `Whitey`。

**Gitee**：[https://gitee.com/dmaker1993/mdResume](https://gitee.com/dmaker1993/mdResume)
**GitHub**：[https://github.com/Dmaker1993/mdResume](https://github.com/Dmaker1993/mdResume)

最后推荐个可以在线编辑markdown简历的网址：[冷熊简历](http://cv.ftqq.com/?fr=github)。

**参考：**

1. [https://dmaker1993.github.io/](https://dmaker1993.github.io/)
2. [https://github.com/Dmaker1993/mdResume](https://github.com/Dmaker1993/mdResume)
3. [https://gitee.com/dmaker1993/mdResume](https://gitee.com/dmaker1993/mdResume)

---

欢迎浏览我的更多文章：[一名不愿透露姓名的程序员](https://dmaker1993.github.io/) 或者  https://dmaker1993.github.io

最后：因小的才疏学浅，如有问题，请不吝指出，感谢感谢～