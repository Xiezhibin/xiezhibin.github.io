# Plugins

## Google java format

- 安装

  - 点击 Marketplace，搜索google-java-format，并且安装；ntelliJ IDEA→Preferences...→Other Settings→google-java-format Settings 点击Enable google-java-format。

  - **My setup**

    - Intellij 2021.3 (Build #IU-213.7172.25, built on March 16, 2022)
    - google-java-format plugin 1.16.0.2
    - Activated "Reformat code" and "Optimize imports" under settings -> Tools -> actions on save
      *I have imported the [intellij-java-google-style](https://raw.githubusercontent.com/google/styleguide/gh-pages/intellij-java-google-style.xml) file under settings -> code style -> Scheme.
    - Java(TM) SE Runtime Environment 18.9 (build 11.0.12+8-LTS-237)
    - VM: OpenJDK 64-Bit Server VM by JetBrains s.r.o.

  - Help→Edit Custom VM Options...

    ```text
    --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
    ```

- 作用

  检查代码是否符合google的代码规范

- 修改缩进空格数量

- 检查

  ```
  执行 checkstyle 检查
  
  执行命令
  
  mvn checkstyle:check
  //或者
  mvn checkstyle:checkstyle
  ```

参考：`https://github.com/google/google-java-format`