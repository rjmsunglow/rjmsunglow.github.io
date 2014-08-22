---
layout: post
title: "如何建立Octopress博客"
date: 2014-08-04 16:29
comments: true
categories: 
---
###前言

一直想找一个能够写自己博客的地方，看到很多人都是用gitHub来生成自己的Blog，然后自己研究了一下如何生成自己的gitHub page，然后发现了一个博客系统Octopress.

Octopress是利用Jekyll博客引擎开发的一个博客系统，生成的静态页面能够很好的在github page上展现。号称是hacker专属的一个博客系统(A blogging framework for hackers.)

下面我就来介绍一下使用Octopress创建自己Blog的步骤：
###1.安装Ruby
Octopress需要Ruby环境，RVM(Ruby Version Manager)负责安装和管理Ruby的环境。所以我们先在终端输入如下命令，来安装RVM：

```
curl -L https://get.rvm.io | bash -s stable --ruby 
```

Octopress需要Ruby 1.9.3的环境，所以我们还需要安装Ruby 1.9.3

```
rvm install 1.9.3  
rvm use 1.9.3  
rvm rubygems latest 
```
###2.安装Octopress
安装Octopress之前需要安装git；由于我之前装过git所以就可以利用git命令将Octopress从github上clone下来；

```
git clone git://github.com/imathis/octopress.git octopress  
cd octopress    # If you use RVM, You'll be asked if you trust the .rvmrc file (say yes).  
ruby --version  # Should report Ruby 1.9.3 
```
然后需要安装相关依赖

```
gem install bundler  
rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command  
bundle install
```
安装完之后需要安装Octopress主题

```
rake install
```
###3.配置Octopress
config.yml是博客重要的一个配置文件，在config.yml文件中有三大配置项：Main Configs、Jekyll & Plugins和3rd Party Settings。
一般，该文件中其中url是必须要填写的，这里的url是在github上创建的一个仓库地址，具体请看第四步中创建的地址。另外再修改一下title、subtitle和author，根据需求，在开启一些第三方组件服务

###4.将博客部署到gitHub上
##### *在github上创建一个repository
   命名规则为 yourName.github.io

  *建立本地与github的联系
  
  ```
  rake setup_github_pages
  ```
  *自定义样式
  博客的基本信息在_config.yml文件中修改. 自定义css等在source文件夹中. 
  完成上面的命令之后，我们就可以生成博客并真正的部署到仓库中了。执行如下命令：
  
  ```
  rake generate
  rake deploy
  ```
###5.发表文章
创建一个新的markdown文件：

 ```
 rake new_post["octopress Blog"]
 ```
 
 这样所创建出来的博客名称是英文的，就算是你刻意写成中文还是会自己转译为英文的，所以如果再去文件里修改title，再修改index.html则会显得很麻烦，无疑是增加了工作量，所以我也找到一个方便的方法来解决这个问题：

 打开在octopress根目录下的Rakefile文件，在文件末尾增加如下代码：
 
 
 ```
 esc "Begin a new post in #{source_dir}/#{posts_dir} with Alias"
task :post, :title, :title_alias do |t, args|
  raise "### You haven't set anything up yet. First run `rake install` to set up an Octopress theme." unless File.directory?(source_dir)
  mkdir_p "#{source_dir}/#{posts_dir}"
  args.with_defaults(:title => 'new-post')
  title = args.title
  title_alias = args.title_alias
  filename = "#{source_dir}/#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title_alias}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end
end
```

新的post任务接收两个参数，第一个参数与之前的new_post任务需要的参数完全相同，第二个参数alias就可以设置到markdown文件的title:中。例如我们要建立一个博客，标题为“如何建立Octopress博客”，则可以输入命令：

```
rake post["How to create octopress blog","如何建立Octopress博客"]
```

注意，除了第二个参数引号内的内容可以用中文外，其他字符包括双引号、逗号和中括号都应该是Utf-8字符。此外，Title和Alias之间用“,”隔开，“,”后不能加空格。

*加入图片在Octopress中，支持的Markdown语法为：

```
{% img [position] [url] [width] [height] %}
```

```
![Smaller icon](http://smallerapp.com/favicon.ico "Title here")
```

我们的博客已经完成基本的部署，不过博客的source需要单独提交，执行如下命令就可以将source提交到仓库的source分支下。

```
git add .  
git commit -m 'Initial source commit'  
git push origin source
```
下面是创建并部署博文的一个完整过程：

```
rake post["How to create octopress blog","如何建立Octopress博客"] 
$ rake generate  
$ git add .  
$ git commit -am "Some comment here."   
$ git push origin source  
$ rake deploy 
```

###6.删除博文
删除文章就很简单了，直接将你写的markdown文件删除之后，执行以下命令即可：

```
rake generate  
rake deploy  
```