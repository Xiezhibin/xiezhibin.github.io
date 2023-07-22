# 个人网址建立

## 网站构建说明

参考：杨希杰的个人网站 [https://yang-xijie.github.io](https://yang-xijie.github.io)中网址构建部分操作。

## 网址文档修改

### 基于 Web 开发和修改个人网址

#### github 基于Web编辑

基于Web的github.dev编辑器，编辑、提交和更改文件。

- [仓库的地址从com修改为dev，使用github.dev在线编译](<https://docs.github.com/zh/codespaces/the-githubdev-web-based-editor>)
- 修改代码
- [设置vscode配置和拓展的同步，这里主要用于和本地或者其它机器的vscode保持一致](https://code.visualstudio.com/docs/editor/settings-sync)
- [使用vscode代码管理器提交代码](https://cloud.tencent.com/developer/article/1793472)

**代码修改**
代码修改一般主要分2个部分，一是修改docs中的文件，二是修改mkdocs.yml中[Navigtion]下的网页关系映射。

修改建议，使用[Typora](https://typoraio.cn)编辑好后，将md文件内容复制粘贴到docs文件中。

**代码管理器提交**
代码提交可以通过vscode的源代码管理（source control）完成

- 添加修改文件
- 提交本地修改项目的git Commit日志，如规范可以参考 [Git | Git Commit 规范](http://119.23.219.145/posts/git-git-commit-%E8%A7%84%E8%8C%83/)中 2.2 小节
- 点击 Commit&Push commit同步内容
- 点击同步提交同步

## MarkDown 文档知识点

文件和文件夹命名：docs文件命名或者文件夹名称不会直接展示在个人网址页面，以英文输入文件夹和文件名称，用下划线，“_”符号来区分两侧单词。

markdown文件规范：[markdown文件建议参考google的规范编写](<https://github.com/google/styleguide/blob/gh-pages/docguide/style.md>)