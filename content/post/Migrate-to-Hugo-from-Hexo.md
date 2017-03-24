+++
date = "2016-09-30T17:14:37+08:00"
description = "从 Hexo 迁移到 Hugo"
title = "Migrate to Hugo from Hexo"
tags = ["hugo", "hexo", "blog"]

+++


把博客从 [Hexo](http://hexo.io/) 迁移到了 [Hugo](http://gohugo.io/)。主要原因有二：

  + Hexo 中文乱码。当中文大概超过1000字，就出现奇怪的乱码。看了 Hexo 的 ISSUE，这个问题也不只我遇到。
  + Hexo 生成静态文件太慢了，等上十几二十秒是常有的事。

遂弃用 Hexo。


## 1. 将 Hexo 的 YAML 转换为 Hugo 的 TOML

Hugo 的配置文件是 TOML 格式的，其生成的 Markdown 文件的 front matter（不知如何翻译）也是 TOML 结构：

```
+++
date = "2016-09-30T17:14:37+08:00"
description = ""
title = "Migrate to Hugo from Hexo"
tags = ["hugo", "hexo", "blog"]

+++
```

而 Hexo 的配置文件是 YAML 格式，其 Markdown 文件的 front matter 如下：

```
title: 'Start React with Webpack'
date: 2016-09-09 04:11:13
tags:
  - webpack
  - react

---
```

所以直接将 Hexo 的 Markdown 文件复制到 Hugo 中是不行的，必须得转换一下格式。虽然 Hugo 也可以直接在 `config.toml` 中配置 `metaDataFormat:"yaml"`，这样 Hugo 就会生成 YAML 结构的 front matter ，但原 Hexo 的 Markdown 文件依旧不能正常的转换，因为 `date` 的格式不一样。

在 Hexo 中，文件里面的 `date: 2016-09-09 04:11:13` 并没有存储时区信息，而是把时区放在了 `_config.yml` 配置文件中。

By the way，TOML 是 GitHub 觉得 YAML 不够简洁优雅，所以捣鼓出来的一个东西。既然现在在用 Hugo，就没有理由不用 TOML 代替原来的 YAML。

其实两者之间的转换也很简单，我用 JS 写了一个脚本来将 YAML 转换为 TOML：

```
// yaml_to_toml.js
const readline = require('readline');
const fs = require('fs');
const os = require('os');
const moment = require('moment-timezone');  // 需要通过 npm install moment-timezone 来安装


const timezone = 'Asia/Shanghai';  // 时区
const src = 'hexo';  // hexo .md 文件源目录
const target = 'post';  // 目标文件目录


// 开始转换
readDir();


// 遍历目录
function readDir() {
    // read all files in src
    fs.readdir(src, function(err, files) {
        files.map((filename) => {
            // get the file extension
            const extension = filename.substr(filename.lastIndexOf('.', filename.length));
            if (extension === '.md') {
              readFile(`${filename}`);
            }
        });
    });

}


function readFile(filename) {
  fs.readFile(`${src}/${filename}`, { encoding: 'utf8' }, function(err, data) {
      if (err) {
          return console.log('err: ', err);
      }

      const content = data.split('---');
      const head = content[0].split('\n');
      // console.log('head: ', head);

      let newHead = head.map((item, index) => {
        // console.log('slpitHead: ', slpitHead(item, index, head));
        return slpitHead(item, index, head);
      });
      newHead = newHead.filter((item) => {return item;});
      // console.log('newHead: ', newHead);
      const newContent = `+++${os.EOL}${newHead.join(os.EOL)}${os.EOL}${os.EOL}+++${os.EOL}${content[1]}`;
      fs.writeFile(`${target}/${filename}`, newContent, {
          encoding: 'utf8'
      }, function(err) {
          if (err) {
            throw err;
          }
          console.log(`${filename}  生成成功！`);
      });
  });
}



function slpitHead(item, index, head) {
  // title
  if (item.indexOf('title:') !== -1) {
    return `title = "${item.split('title:')[1].trim()}"`;
  }

  // date
  if (item.indexOf('date:') !== -1) {
    return `date = "${(moment.tz(item.split('date:')[1], timezone)).format()}"`;
  }

  // tags
  if (item.indexOf('tags:') !== -1) {
    // console.log('tags...');
    const tags = [];
    for (let i=index+1; i<head.length; i++) {
      if (head[i].indexOf('-') !== -1) {
        // console.log('head[i].split('-')[1]: ', head[i].split('-')[1]);
        tags.push(head[i].split('-')[1].trim());
      } else {
        break;
      }
    }
    // console.log('tags: ', tags);
    return `tags = ${JSON.stringify(tags)}`;
  }

  // categories
  if (item.indexOf('categories:') !== -1) {
    const categories = [];
    for (let i=index+1; i<head.length; i++) {
      if (head[i].indexOf('-') !== -1) {
        categories.push(head[i].split('-')[1].trim());
      } else {
        break;
      }
    }
    // console.log('categories: ', categories);
    return `categories = ${JSON.stringify(categories)}`;
  }

  return false;
}
```


先配置好 `timezone` `src` `target` 三个参数。然后 `npm install moment-timezone` 安装需要的第三方包，最后 `node yaml_to_toml.js` 即可。


## 2. 图片目录的迁移

本地图片迁移也非常简单。

我之前使用 Hexo 的时候，是将所有图片都放在 `source/images/` 目录里面的，在 Markdown 文件中引入图片是这样的： `![image title](/iamges/image_name.png)`。

Hugo 的图片可以直接放在其 `static/` 目录里面，其路径就是 `/iamges/image_name.png`，所以我只需要将 Hexo 中的 `images` 目录复制到 Hugo 的 `static/` 目录下即可。


## 3. 主题

看遍了 Hugo 给出的所有主题，都不满意。很多主题都超级简洁，这种风格还是很喜欢的。所以决定自己写一个。

最终 Fork 了 [https://github.com/digitalcraftsman/hugo-cactus-theme](https://github.com/digitalcraftsman/hugo-cactus-theme) 这个主题，然后改成了自己想要的样子。

主页

![Migrate-to-Hugo-from-Hexo-1.png](http://oh1ywjyqf.bkt.clouddn.com/blog/2016-11-22-Migrate-to-Hugo-from-Hexo-1.png)

文章页

![Migrate-to-Hugo-from-Hexo-2.png](http://oh1ywjyqf.bkt.clouddn.com/blog/2016-11-22-Migrate-to-Hugo-from-Hexo-2.png)

改后的主题源码：[https://github.com/nodejh/hugo-cactus-theme](https://github.com/nodejh/hugo-cactus-theme)。


## 4. 部署

由于 Github Pages 国内访问速度慢，所以同时把静态页面部署到了 Github Page 和 Coding Pages。


```
// 添加两个仓库
git remote add all https://github.com/nodejh/nodejh.github.io
git remote set-url origin --push --add https://git.coding.net/nodejh/nodejh.git
git remote set-url origin --push --add https://github.com/nodejh/nodejh.github.io
```

然后只需要执行 `git push all branch` 就可以同时向两个仓库 push 代码了。但暂时不这样做。而是在 `public` 目录外创建一个 `deploy.sh` 目录，用来自动部署：

```
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace by `hugo -t <yourtheme>`

# Go To Public folder
cd public
# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back
cd ..
```

接下来再给 `deploy.sh` 添加可执行权限 `chmod +x deploy.sh`。然后每次写完东西只需要 `.／deploy.sh 'commit message'` 即可。


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/11](https://github.com/nodejh/nodejh.github.io/issues/11)
