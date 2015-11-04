title: EditorConfig
date: 2015-11-04 14:06:07
tags:
---


[editorconfig](http://editorconfig.org/)是一款帮助开发者在不同的ide和编辑器之间，定义和维护编码格式的工具。

intellij是native support for editor config，但是绝大部分ide都有插件支持。

<!--more-->

## 配置文件的加载

当打开一个文件时，editor config会去在打开文件所在的目录，找一个`.editorconfig`的文件，并且editor config还会去一级一级地查找这个目录的所有父目录。当查找到根目录，或者找到一个`.editorconfig`文件，在这个文件中定义了`root=true`。

EditorConfig会根据目录的层级，从上往下读所有的`.editorconfig`，所以离打开文件最近的EditorConfig files最后读取，配置的属性会根据读取的属性进行覆盖，所以离打开文件越近的配置文件，具有更高的优先级。

windows用户不能直接创建`.editorconfig`, 那么直接创建一个`.editorconfig.`文件，windows explorer会自动将他命名为`.editorconfig`。


## 配置

EditorConfig的配置可以参考它的官网，这里只是举一个java项目中的例子, 基本和官网的例子相同。

```
# EditorConfig is awesome: http://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true


# Matches multiple files with brace expansion notation
# Set default charset
[*.{js,py,java,html,xml}]
charset = utf-8

# 4 space indentation
[*.{py,java}]
indent_style = space
indent_size = 4

# Tab indentation (no size specified)
[Makefile]
indent_style = tab

# Indentation override for all JS under lib directory
[**.{js,html,xml}]
indent_style = space
indent_size = 2

# Matches the exact files either package.json or .travis.yml
[{package.json,.travis.yml}]
indent_style = space
indent_size = 2

[*.md]
indent_size = 4

```

具体每个字段可以参考：
[EditorConfig Properties](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties)

## eclipse 插件

eclipse请下载这个插件。
https://github.com/ncjones/editorconfig-eclipse#readme



## reference

[use-editorconfig-to-manage-coding-styles-on-team-projects](https://viget.com/extend/use-editorconfig-to-manage-coding-styles-on-team-projects)
[why-i-use-editorconfig](ttp://davidensinger.com/2013/07/why-i-use-editorconfig/)
[use-editorconfig-to-maintain-consistent-coding-styles-between-different-editors-and-ides](https://www.topbug.net/blog/2012/03/14/use-editorconfig-to-maintain-consistent-coding-styles-between-different-editors-and-ides/)
