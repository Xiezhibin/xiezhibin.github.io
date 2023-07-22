# 插件

## Google Java Format

* **安装**
  * **在IntelliJ IDEA中，点击“Marketplace”，搜索并安装“google-java-format”插件。然后在IntelliJ IDEA的设置中启用google-java-format插件。路径：IntelliJ IDEA → Preferences... → Other Settings → google-java-format Settings，勾选“Enable google-java-format”选项。**
  * **我的配置**
    * **IntelliJ IDEA 2021.3（版本号：IU-213.7172.25，构建日期：2022年3月16日）**
    * **google-java-format 插件版本：1.16.0.2**
    * **在设置中激活了“保存时重新格式化代码”和“优化导入”选项。路径：设置 -> 工具 -> 在保存时执行的操作。**
    * **在设置中导入了 **[intellij-java-google-style](https://raw.githubusercontent.com/google/styleguide/gh-pages/intellij-java-google-style.xml) 文件，并将其作为代码风格方案。路径：设置 -> 代码样式 -> 方案。
    * **Java(TM) SE Runtime Environment 18.9（版本号：11.0.12+8-LTS-237）**
    * **VM：JetBrains s.r.o. 提供的 OpenJDK 64位服务器虚拟机。**
  * **编辑自定义VM选项：帮助 → 编辑自定义VM选项...**
    ```
    --add-exports=jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED
    --add-exports=jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED
    ```
* **作用**
  **google-java-format插件用于检查代码是否符合Google的代码规范，并修改代码的缩进和空格数量，使其符合规范。**
* **检查**
  **可以通过执行checkstyle检查来验证代码是否符合Google的代码规范。执行以下命令：**
  ```
  mvn checkstyle:check
  //或者
  mvn checkstyle:checkstyle
  ```

**参考链接：**`https://github.com/google/google-java-format`
