---
title: 'HEXO高亮亮不起来的解决方法'
date: 2016-12-19 23:41:20
tags: Web
---

刚刚进入HEXO的坑，发现这东西确实是个神器，全静态网页，简单，方便，支持markdown更新文章，很方便。在调教hexo的过程中也遇到很多坑，尤其是代码高亮的问题，一开始让我非常头疼。作为一个前端小白，尝试了很多方法，查询了很多博客，也没有找到解决方案。许多博主大神写的比较easy，没说清楚。我现在详细叙述一下我的解决方法。
 ![hexo](https://p1.bqimg.com/567571/74dd2bc9dbdc0c5e.jpg)

<!--more-->

* 首先需要安装一个插件[*Hexo-Prism-Plugin*](https://github.com/ele828/hexo-prism-plugin)：

```sh
npm i -S hexo-prism-plugin
```

* 然后编辑hexo主目录下的*_config.yml*,添加以下字段：

```sh
prism_plugin:
  mode: 'preprocess'    # realtime/preprocess
  theme: 'default'
```

  realtime：在加载网页的时候解析高亮代码
  
  preprocess: 在nodejs中预处理加载
  
  theme:
  
   1. default 
   2. coy
   3. dark
   4. funky
   5. okaidia
   6. solarizedlight
   7. tomorrow
   8. twilight
  
* 接下来下就clean & generate

```sh
hexo clean && hexo g
```

* 然后在md文档中采用以下格式插入代码即可

\`\`\`+language

 code
 
 \`\`\`
 
 
