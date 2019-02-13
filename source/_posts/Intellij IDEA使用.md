---
title: Intellij IDEA 使用
date: 2019-02-13 17:27:28
updated: 2019-02-13 17:27:28
categories: Intellij IDEA
tags: Intellij IDEA

---

# Intellij IDEA使用

1. 关闭自动更新

   Intellij IDEA -> Preferences -> Appearance & Behavior -> System Settings -> Updates 下取消 Automatically check updates for勾选

2. 代码编辑器主题风格

   Intellij IDEA -> Preferences -> Editor -> Colors & Fonts -> Font

   ```powershell
   Scheme: Darcula
   Show only monospaced fonts 设置第一字体，Monaco 不支持中文
       Primary font:Monaco Size:20 Line spacing:1.0
   Secondary font：YaHei Consolas Hybrid  设置第二字体
   ```

3. 文件编码

   File -> Settings -> Editor -> File Encodings

   ```powershell
   Global Encoding:UTF-8
   Projectt Encoding:UTF-8
   Default encoding for properties files:UTF-8
   勾选上Transparent native-to-ascii conversion
   ```

4. 类和方法注视模版

   在File -> Settings -> Editor -> File and Code Templates

5. 编码缩进

   Intellij IDEA -> Preferences -> Editor -> Code Style -> Erlang

   Use tab character 不要勾选，然后indent设置为4，代表按一个tab为4个空格，并且自动整理格式也是4个空格一缩进。

   ```powershell
   Tab size: 4 
   Indent: 4
   Continuation indent: 8
   ```

   如果要对多个文件进行转换，可以在文件夹上面按右键，然后点击Reformat Code或者选中文件夹按快捷键ctrl+alt+L对多个快捷键整理。
