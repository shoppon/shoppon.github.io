---
title: "vscode配置"
categories: ["vscode"]
tags: ["vscode"]
date: 2020-02-28T17:05:23+08:00
---

# vscode配置

```json
{
    "[c]": {
        "editor.defaultFormatter": "xaver.clang-format"
    },
    "[cpp]": {
        "editor.defaultFormatter": "xaver.clang-format"
    },
    "[dockerfile]": {
        "editor.defaultFormatter": "ms-azuretools.vscode-docker"
    },
    "[html]": {
        "editor.defaultFormatter": "vscode.html-language-features"
    },
    "[java]": {
        "editor.defaultFormatter": "xaver.clang-format"
    },
    "[javascript]": {
        "editor.defaultFormatter": "vscode.typescript-language-features"
    },
    "C_Cpp.default.includePath": [
        "${workspaceFolder}",
        "${workspaceFolder}/**",
        "/usr/local/dra/include",
        "/usr/local/include",
        "/usr/local/lib/gcc/x86_64-pc-linux-gnu/7.3.0/include"
    ],
    "clang-format.executable": "C:/Program Files/LLVM/bin/clang-format.exe",
    "cmake.configureOnOpen": true,
    "code-runner.executorMap": {
        "python": "python3"
    },
    "code-runner.runInTerminal": true,
    "docker.containers.label": "ContainerId",
    "doxdocgen.file.copyrightTag": [
        "Copyright (c) shopppon@gmail.com"
    ],
    "doxdocgen.file.fileOrder": [
        "copyright",
        "empty",
        "file",
        "author",
        "brief",
        "version",
        "date",
        "empty",
        "custom"
    ],
    "doxdocgen.generic.authorEmail": "xiaopeng2@huawei.com",
    "doxdocgen.generic.authorName": "x00250203",
    "doxdocgen.generic.authorTag": "@author {author}",
    "editor.multiCursorModifier": "ctrlCmd",
    "editor.renderWhitespace": "all",
    "editor.suggestSelection": "first",
    "editor.tabCompletion": "on",
    "editor.wordWrap": "on",
    "explorer.confirmDelete": false,
    "explorer.confirmDragAndDrop": false,
    "extensions.ignoreRecommendations": true,
    "files.associations": {
        "*.py": "python"
    },
    "files.autoGuessEncoding": true,
    "files.eol": "\n",
    "files.exclude": {
        "**/__pycache__": true,
        "**/.classpath": true,
        "**/.factorypath": true,
        "**/.project": true,
        "**/.settings": true,
        "**/*.pyc": true
    },
    "files.insertFinalNewline": true,
    "files.trimFinalNewlines": true,
    "files.trimTrailingWhitespace": true,
    "git.ignoreLegacyWarning": true,
    "gitlens.advanced.messages": {
        "suppressGitVersionWarning": true
    },
    "python.jediEnabled": false,
    "shellformat.path": "/usr/bin/shfmt",
    "tabnine.experimentalAutoImports": true,
    "terminal.integrated.shell.linux": "/usr/bin/bash",
    "todo-tree.highlights.enabled": true,
    "vim.leader": ",",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "window.zoomLevel": 0,
    "workbench.colorTheme": "One Dark Pro"
}


```
