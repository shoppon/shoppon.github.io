```
title: "从gitbook迁移到hugo"
categories: ["python"]
tags: ["gitbook", "hugo"]
date: 2020-10-22T14:43:03+08:00
```

# 背景

之前的文章一直是用`gitbook`管理的，`gitbook`又是基于`nodejs`的，众所周知的原因`npm`源在国内特别慢，而且`nodejs`本身也很慢。而且`gitbook`在高版本的`nodejs`中无法运行，基于这两点打算将博客迁移到其他平台。

在所有的博客平台中，最终在`hexo`和`hugo`之间纠结。`hexo`的生态比较丰富，插件、主题比较多，`hugo`是用`go`写的，性能较高。

由于对`go`一直比较有兴趣，而且其运行速度较快，最终决定迁移到`hugo`平台。

# gitbook与hugo的差异

两者都是用`markdown`进行写作，`gitbook`是通过目录进行分类管理，而`hugo`是通过在文件头添加摘要的方式进行元数据（分类、标题、标签、时间）管理。

`gitgook`通过一个`SUMMARY.md`的文件定义章节结构，博客标题和分类在列表中定义。从迁移过程中，需要将文档所在目录作为分类，将第一个一级标题作为文章标题，时间需要从`git`提交中获取文档的创建时间。

# 工具

这种规则清晰、大量重复的事情最好是教给计算机来实现，上百篇文章不可能靠人工迁移。而`python`又是最适合这种场景编程的语言，书写方便，API丰富，易于调试。

用`google`搜索了下，可以用`git log --diff-filter=A --follow --format=%aI {filename}'`命令获取git仓库中文档创建时间。用`os.walk`函数可以遍历所有文件，然后读取其目录和标题，最后写入到文件的摘要中。

完整代码如下：

```python
import subprocess
import sys
import os


def get_created_at(root, filename):
    sub = subprocess.Popen(f'git log --diff-filter=A --follow --format=%aI {filename}', shell=True, stdout=subprocess.PIPE,
                           cwd=root)
    ret = sub.communicate()
    return str(ret[0], encoding='utf-8').rstrip('\n')


def get_category(filename):
    return os.path.basename(os.path.dirname(os.path.abspath(filename)))


def get_title(f):
    f.seek(0)
    for line in f:
        if line.startswith("# "):
            return line[2:].rstrip('\n')
    return "Unknown"


def generate_digest(root, filename):
    real_path = os.path.join(root, filename)
    tmp_path = os.path.join(root, 'tmp_file.md')
    created = get_created_at(root, filename)
    category = get_category(real_path)
    with open(real_path, 'r') as f:
        first_line = f.readline()
        if first_line.startswith('---'):
            return
        with open(tmp_path, 'w') as f2:
            title = get_title(f)
            f2.write(f'---\n')
            f2.write(f'title: "{title}"\n')
            f2.write(f'categories: ["{category}"]\n')
            f2.write(f'tags: [""]\n')
            f2.write(f'date: {created}\n')
            f2.write(f'---\n')
            f2.write('\n')
            f.seek(0)
            f2.write(f.read())
    os.rename(tmp_path, real_path)


if __name__ == '__main__':
    for root, dirs, files in os.walk(os.path.dirname(os.path.abspath("your path"))):
        for f in files:
            if not f.endswith('.md'):
                continue
            generate_digest(root, f)

```

# 参考

- [Finding the date/time a file was first added to a Git repository](https://stackoverflow.com/questions/2390199/finding-the-date-time-a-file-was-first-added-to-a-git-repository)

