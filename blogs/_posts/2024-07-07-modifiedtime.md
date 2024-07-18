---
layout: post
title: 使用命令行，一键修改文件夹所有文件修改日期改为最新
---

可以使用操作系统自带的命令行工具或者脚本来批量修改文件夹里面所有文件的修改日期。以下是两种常用的方法：

## 在 Windows 系统中使用 PowerShell

在 Windows 系统中，可以使用 PowerShell 来批量修改文件夹里面所有文件的修改日期。具体步骤如下：

1. 打开 PowerShell 工具。在 Windows 10 中，可以按下 Win + X 键，选择“Windows PowerShell”或“Windows PowerShell (管理员)”。
2. 切换到需要修改日期的文件夹目录。可以使用 cd 命令切换到指定目录，例如 cd D:\files。
3. 执行以下命令来修改文件夹里面所有文件的修改日期为当前日期：

```sh
Get-ChildItem -recurse | ForEach-Object { $_.LastWriteTime = Get-Date }
```

上述命令中，-recurse 参数表示递归遍历文件夹及其子文件夹中的所有文件。$_ 表示当前文件的对象，LastWriteTime 属性表示文件的修改日期。

## 在 Linux/MacOS 系统中使用 Shell 脚本

在 Linux/MacOS 系统中，可以使用 Shell 脚本来批量修改文件夹里面所有文件的修改日期。具体步骤如下：

1. 创建一个名为 `change_file_date`.sh 的 Shell 脚本文件，并将其保存到需要修改日期的文件夹中。可以使用 touch 命令创建该文件，例如 `touch change_file_date.sh`。
2. 打开 change_file_date.sh 文件，并输入以下内容：

```sh
#!/bin/bash
for file in $(find . -type f)
do
  touch "$file"
done
```

上述代码中，我们使用 find 命令查找当前文件夹及其子文件夹中的所有文件，然后使用 touch
