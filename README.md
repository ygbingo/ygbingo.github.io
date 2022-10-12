# ygbing.github.io
Personal Pages

## 步骤

### Hugo

1. 安装Hugo(以Mac为例)

   ```bash
   $ brew install hugo
   ```

2. 创建本地文件夹，用于存储博客内容，假设文件夹叫blog

   ```bash
   $ hugo new site blog
   ```

3. blog目录下的文件内容如下：

   ```markdown
   drwxr-xr-x   9 xxx  staff  288 12 21 14:46 .
   drwxr-xr-x  17 xxx  staff  544 12 21 14:46 ..
   drwxr-xr-x   3 xxx  staff   96 12 21 14:46 archetypes
   -rw-r--r--   1 xxx  staff   82 12 21 14:46 config.toml
   drwxr-xr-x   2 xxx  staff   64 12 21 14:46 content
   drwxr-xr-x   2 xxx  staff   64 12 21 14:46 data
   drwxr-xr-x   2 xxx  staff   64 12 21 14:46 layouts
   drwxr-xr-x   2 xxx  staff   64 12 21 14:46 static
   drwxr-xr-x   2 xxx  staff   64 12 21 14:46 themes
   ```

### even主题

给Hugo配置even主题：

1. 下载主题到blog下的themes目录，在blog所在目录执行如下命令：

   ```bash
   $ git clone https://github.com/olOwOlo/hugo-theme-even themes/even
   ```

2. 使用even主题自带的全局配置config.toml覆盖Hugo初始安装的config.toml

   ```bash
   $ cp themes/even/exampleSite/config.toml ./
   ```

3. 根据个人需要，修改blog/config.toml里的配置项，比如网站首页title，右上角菜单名称，底部链接名称等，可以参考[我的配置](https://github.com/jincheng9/blog)。

   **注意**：highlightInClient配置项和pygments开头的配置项不能都启用，都启用会导致markdown里插入代码块的时候样式错误。highlightInClient使用默认的false即可，不用修改。

4. 使用even主题自带的博客文章默认配置`themes/even/archetypes/default.md`覆盖Hugo初始安装的`archetypes/default.md`，在blog根目录执行如下命令：

   ```bash
   cp themes/even/archetypes/default.md ./archetypes/
   ```

   修改`archetypes/default.md`里的默认配置项，把comment和toc都设置为true，用于开启文章评论功能和文章自动生成目录功能，可以参考[我的配置](https://github.com/jincheng9/blog)。

5. 启动本地Hugo server，查看博客效果，本地地址：http://localhost:1313/

   ```bash
   $ hugo
   $ hugo server
   ```



### 写文章和发布

1. 在blog目录，使用`hugo new post/some-content.md`创建markdown文件

   ```bash
   $ hugo new post/test.md
   ```

2. 发布：在blog根目录执行`hugo`命令或者`hugo -t even`命令，自动根据md文件生成文章的静态页面，静态页面发布在根目录的public文件夹下面

   ```bash
   $ hugo
   ```

3. 启动Hugo server，查看网站最新效果，在blog根目录执行如下命令：

   ```bash
   $ hugo server
   ```


# Q&A
1. git actions 失败，提示"No url found for submodule path 'themes/even' in .gitmodules"
新建并编辑```.gitmodules```
``` shell
[submodule "themes/even"]
  path = themes/even
  url = git@github.com:olOwOlo/hugo-theme-even.git
```