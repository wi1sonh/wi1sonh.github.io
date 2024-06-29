---
layout: post
title: tmux cheatsheet
---

tmux牛逼

## 前缀键

所有快捷键都要通过前缀键(默认是Ctrl+b)唤起，帮助命令的快捷键是`Ctrl+b ?`（在 Tmux 窗口中，先按下Ctrl+b，再按下?，就会显示帮助信息，按下 ESC 键或 q 键，就可以退出帮助）

## 会话

**新建会话**: `tmux new -s <session-name>`，若不指定则默认编号0,1,2...

**分离会话**: 在 Tmux 窗口中，按下`Ctrl+b d`或者输入`tmux detach`命令，就会将当前会话与窗口分离

**查看当前所有的 Tmux 会话**: `tmux ls/list-session`

**接入会话**: `tmux a/attach -t id/<session-name>`

**杀死会话**: `tmux kill-session -t id/<session-name>`

**切换会话**: `tmux switch -t id/<session-name>`

**重命名会话**: `tmux rename/rename-session -t 0 <new-name>`

- `Ctrl+b d`：分离当前会话
- `Ctrl+b s`：列出所有会话
- `Ctrl+b $`：重命名当前会话
- `Ctrl+b [ / PageUp`：复制模式，可以自由上下滚动，按g并输入行号跳转，按ESC退出

`.tmux.conf`：set -g mouse on

## 窗格

**划分窗格**:

```sh
# 划分上下两个窗格
$ tmux split-window

# 划分左右两个窗格
$ tmux split-window -h
```

**移动光标**:

```sh
# 光标切换到上方窗格
$ tmux select-pane -U

# 光标切换到下方窗格
$ tmux select-pane -D

# 光标切换到左边窗格
$ tmux select-pane -L

# 光标切换到右边窗格
$ tmux select-pane -R
```

**交换窗格位置**:

```sh
# 当前窗格上移
$ tmux swap-pane -U

# 当前窗格下移
$ tmux swap-pane -D
```

- `Ctrl+b %`：划分左右两个窗格。
- `Ctrl+b "`：划分上下两个窗格。
- `Ctrl+b <arrow key>`：光标切换到其他窗格。`<arrow key>`是指向要切换到的窗格的方向键，比如切换到下方窗格，就按方向键↓。
- `Ctrl+b ;`：光标切换到上一个窗格。
- `Ctrl+b o`：光标切换到下一个窗格。
- `Ctrl+b {`：当前窗格与上一个窗格交换位置。
- `Ctrl+b }`：当前窗格与下一个窗格交换位置。
- `Ctrl+b Ctrl+o`：所有窗格向前移动一个位置，第一个窗格变成最后一个窗格。
- `Ctrl+b Alt+o`：所有窗格向后移动一个位置，最后一个窗格变成第一个窗格。
- `Ctrl+b x`：关闭当前窗格。
- `Ctrl+b !`：将当前窗格拆分为一个独立窗口。
- `Ctrl+b z`：当前窗格全屏显示，再使用一次会变回原来大小。
- `Ctrl+b Ctrl+<arrow key>`：按箭头方向调整窗格大小。
- `Ctrl+b q`：显示窗格编号。

## 窗口

**新建窗口**:

```sh
$ tmux new-window

# 新建一个指定名称的窗口
$ tmux new-window -n <window-name>
```

**切换窗口**:

```sh
# 切换到指定编号的窗口
$ tmux select-window -t <window-number>

# 切换到指定名称的窗口
$ tmux select-window -t <window-name>
```

**重命名窗口**:

```sh
$ tmux rename-window <new-name>
```

- `Ctrl+b c`：创建一个新窗口，状态栏会显示多个窗口的信息
- `Ctrl+b p`：切换到上一个窗口（按照状态栏上的顺序）
- `Ctrl+b n`：切换到下一个窗口
- `Ctrl+b <number>`：切换到指定编号的窗口，其中的`<number>`是状态栏上的窗口编号
- `Ctrl+b w`：从列表中选择窗口
- `Ctrl+b ,`：窗口重命名

## Others

```sh
# 列出所有快捷键，及其对应的 Tmux 命令
$ tmux list-keys

# 列出所有 Tmux 命令及其参数
$ tmux list-commands

# 列出当前所有 Tmux 会话的信息
$ tmux info

# 重新加载当前的 Tmux 配置
$ tmux source-file ~/.tmux.conf
```

## Reference

1. https://hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/
2. https://danielmiessler.com/p/tmux/
3. https://linuxize.com/post/getting-started-with-tmux/
4. https://www.ruanyifeng.com/blog/2019/10/tmux.html
