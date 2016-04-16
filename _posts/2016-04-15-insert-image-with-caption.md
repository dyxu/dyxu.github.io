---
layout: post
title: github pages中插入带caption的图片
author: dyxu
keywords: 
description:
categories:
  - 工具
tags: ["博客", "工具"]

---
{% include JB/setup %}
### 1 最终效果

本文讨论一种在github pages中插入带caption图片的实现方式，最终的效果如图一所示

{% include image.html
            img="images/2016/04/duang.jpg"
            title="最终效果"
            caption="图一 最终效果"
            url="http://dyxu.github.io" %}

### 2 实现方法

-------------------------

#### 2.1 _includes中插入image.html模板

在_includes目录下创建image.html文件，写入以下内容
 
{% include image.html
            img="images/2016/04/image_html.jpg"
            title="image.html模板"
            caption="图二 image.html代码" %}

#### 2.2 加入CSS样式

将图片显示的CSS样式加入网站的样式中，我的目录为assets/css/style.css，内容为(按自己喜好修改)

    .image-wrapper {
        text-align: center;

        .image-caption {
            color: $grey-color;
            margin-top: $spacing-unit / 3;
        }
    }

#### 2.3 应用样式

在文章需要插入图片的位置，用以下方式插入图片即可

{% include image.html
    img="images/2016/04/insert_image.jpg"
    title="插入图片例程"
    caption="图三 插入图片例程" %}

