---
layout: post
title: "使用jekyll+Github Pages搭建个人博客"
categories: "其他"
---

从2号决定要做博客，到现在，总共折腾了两天半，总算把这个博客整好了，这篇文章算我对这个过程的一个总结。

## 准备工作

1. 安装Git，熟悉Git的常用操作（必须）
2. 注册一个Github账号，熟悉Github的操作（必须）
3. 注册一个多说账号，可以为你的博客增加评论功能（可选）

## 开始搭建个人博客

1. 安装jekyll，这个网上教程很多，基本上就是先安装ruby，在安装gem，然后安装jekyll。此处就略过了，大家可以在网上找教程。
推荐：[http://jekyll.bootcss.com/](http://jekyll.bootcss.com/)
2. 寻找jekyll模板，当然，有能力的可以自己制作模板，推荐几个寻找模板的好地方：
    * [jekyllthemes](http://jekyllthemes.org/)，我的模板就是在这找的
    * [知乎：有推荐的简洁明快的jekyll模板吗？](http://www.zhihu.com/question/20223939)
3. 把找好的模板源码下载下来，然后在Github上创建一个空项目，把模板源码上传上去，下面以我用的模板Pithy说明一下具体的过程：
    * 下载模板源码，如下图点击Download下载源码并解压

    ![image](http://oldblog.shicishuzhai.com/2015-10-04_16-46-21.jpg)

    * 登录Github，创建一个空项目，取名为：<你的用户名>.github.io
    * 然后把这个空项目clone到本地，把下载好的模板源码拷进去，然后再push到Github上，完成后的效果如下图

    ![image](http://oldblog.shicishuzhai.com/2015-10-04_16-54-30.jpg)

4. 修改模板内容，主要有以下几点
    * 修改_config.yml，具体修改方法可以参照 [http://higrid.net/c-art-blog_jekyll.htm](http://higrid.net/c-art-blog_jekyll.htm#--3)
    * 修改多说账号，这个一般在_layouts/post.html文件中，有的模板中集成了多说，你只需要更改一下shortname即可，有的模板中没有集成多说，添加如下图的代码即可

    ![image](http://oldblog.shicishuzhai.com/2015-10-04_17-10-06.jpg)

    * 删除_posts文件夹下的内容，这个文件夹下存放的就是你要发表的博客，大多数模板在这个文件夹中都会有几篇博客，你可以删除他们，
    添加你自己的博客，注意文件名的命名规范：yyyy-MM-dd-博客标题.md，博客的头（两个---之间的内容）可以参考模板自带的博客，
    把title改为自己博客的title
5. 测试博客，此时，博客就已经搭建好了，你可以在_posts文件夹中添加新的博客
    * 打开终端，进入博客目录，输入命令`jekyll server -w`
    * 打开浏览器，输入`http://localhost:4000/`，这样就可以看到你的博客啦

    ![image](http://oldblog.shicishuzhai.com/2015-10-04_17-25-46.jpg)
6. 把刚才的修改push到Github上，这样你就有了一个域名为：<你的用户名>.github.io的博客

## 其他的一些优化

* 添加Rakefile
    * 用常规的方法创建博客，需要在_posts文件夹下创建一个.md文件，还要符合相应的命名规范，还要手动添加文件的头，非常麻烦，
        使用Rakefile可以直接通过命令`rake post title="<博客标题>"`，直接创建一篇博客，非常便捷
    * 很多模板不带有Rakefile, 你可以从[我的博客仓库](https://github.com/liujinlongxa/liujinlongxa.github.io)里下载Rakefile文件，
    拷贝到自己博客的根目录下

* 定制博客文件头：根据自己的需求，修改Rakefile，定制博客的头，下面是我定制的内容，默认支持标题和分类，命令格式:
    `rake post title="<博客标题> categories="<博客分类>"`

```shell

    # Usage: rake post title="A Title" [categories="post categories"] [date="2012-02-09"]
    desc "Begin a new post in #{CONFIG['posts']}"
    task :post do
      abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
      title = ENV["title"] || "无题"
      categories = ENV["categories"] || "其他" # 添加分类参数，默认为“其他”分类
      # slug = title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
      slug = title.gsub(' ', '') # 修改博客标题，使其支持中文
      begin
        date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
      rescue Exception => e
        puts "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
        exit -1
      end
      filename = File.join(CONFIG['posts'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
      if File.exist?(filename)
        abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
      end

      puts "Creating new post: #{filename}"

    # 下面就是具体的博客头的内容
      open(filename, 'w') do |post|
        post.puts "---"
        post.puts "layout: post"
        post.puts "title: \"#{title.gsub(/-/,' ')}\""
        post.puts "categories: \"#{categories.gsub(/-/,' ')}\""
        post.puts "---"
      end
    end # task :post
```

* 简化创建博客的命令：每次创建博客输入`rake post title="<博客标题> categories="<博客分类>"`这么一长串命令还是有些繁琐，可以在`~/.zshrc`
    文件（如果没有装Oh-my-zsh，应该是`~/.bashrc`）添加如下代码，这样，以后你就可以直接输入命令 `rpt "<博客标题>" "<博客分类>"`创建博客，
    怎么样，是不是非常方便

```shell
rpt() {
        rake post title=$1 categories=$2
}
```

* 使用七牛存储图片。博客空间是有限的（200M），建议将博客中的图片上传到第三方图床上（这里推荐七牛），博客中直接使用图片的外链，
节省博客空间。

## 接下来要做的

* 目前，我写博客使用的是Atom，平时都是用Xcode，这种编辑器还是第一次用，感觉它对jekyll的markdown支持的不是很好，特别是
`% highlight objectiveC %`代码块，打算以后看看有什么解决办法，不行的话就换个编辑器
* 当前博客使用的还是github的域名，打算以后买个独立域名
* 最后，当然是开始写博客啦！！！！
