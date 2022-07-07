# ygbing.github.io
Personal Pages

# Q&A
1. git actions 失败，提示"No url found for submodule path 'themes/even' in .gitmodules"
新建并编辑```.gitmodules```
``` shell
[submodule "themes/even"]
  path = themes/even
  url = git@github.com:olOwOlo/hugo-theme-even.git
```