title: mathjax in hexo
date: 2015-08-12 14:50:08
tags:
categories: math
math: true
---

最近将博客从octopress迁移到了hexo。
如果使用hexo默认的markdown解析器，然后在用mathjax进行公式显示，存在字符转义的问题。

默认的markdown解析器会将latex中的下划线和'\'进行转义。公式不能正常显示，只有手动转义。

然后在网上找到一篇文章：
[Play MathJax with hexo-renderer-pandoc](http://blog.yuanbin.me/posts/2014/05/play-mathjax-with-pandoc.html)

介绍使用[pandoc](http://pandoc.org/)和[hexo-renderer-pandoc](https://github.com/wzpan/hexo-renderer-pandoc)

## 步骤

1. Firstly, make sure you have [installed pandoc](http://pandoc.org/installing.html).
2. Secondly, cd into your hexo root folder and execute the following command:

```bash
$ npm install hexo-renderer-pandoc --save
```

3. Create a file named mathjax.ejs in the folder like theme_folder/layout/_partial/, the code in mathjax.ejs are as follows:
{% gist 3e2804c717c91a75aa55 mathjax.ejs %}

4. Add the snippits below in theme_folder/layout/_partial/after_footer.ejs
```
<% if (page.mathjax){ %>
<%- partial('mathjax') %>
<% } %>
```

5. Add mathjax: false for _config.yml in the root directory of your Hexo init folder. It is some thing like this:
```
mathjax: false
```

6. For the post you want to load MathJax, add `mathjax: true` at the front-matter (文章开始的头部). By this method you can avoid unnecessary latency for loading Mathjax scripts. Just some thing as follows:
```
mathjax: true
```

7. Delete the public folder and excute `hexo generate` again to make the modified Hexo theme work.

## example

```
The **characteristic polynomial** $\chi(\lambda)$ of the $3 \times 3$ matrix
$$
\left( \begin{array}{ccc}
a & b & c \\
d & e & f \\
g & h & i
\end{array} \right)
$$
is given by the formula
$$
\chi(\lambda) = \left| \begin{array}{ccc}
\lambda - a & -b & -c \\
-d & \lambda - e & -f \\
-g & -h & \lambda - i
\end{array} \right|.
$$
```

The **characteristic polynomial** $\chi(\lambda)$ of the $3 \times 3$ matrix

$$
\left( \begin{array}{ccc}
a & b & c \\
d & e & f \\
g & h & i
\end{array} \right)
$$

is given by the formula

$$
\chi(\lambda) = \left| \begin{array}{ccc}
\lambda - a & -b & -c \\
-d & \lambda - e & -f \\
-g & -h & \lambda - i
\end{array} \right|.
$$