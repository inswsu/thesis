# A model for studying the entrainment of oscillations in a pulse-modulated control system
> 关于研究脉冲调制控制系统中振荡夹带的模型
##2 ./build-Ant-system.sh文件





##1 ./star.sh文件
> # 目的：在具体执行之前，需要了解AnT和libhormo是什么，因为这些参数对于理解代码的具体作用至关重要。如果它们是特定应用程序的组成部分，这个脚本可能用于开启一个服务器实例和多个客户端实例来进行通信或处理数据。
> 这段代码是一个Bash脚本，通常在Linux环境下运行。这里是逐行的注释：
\#!/bin/bash
\# 上面这一行称为shebang，它告诉系统这个脚本应该用哪个解释器来执行，这里指定了bash解释器。

pwd
\# 'pwd' 命令打印出当前工作目录的完整路径。

gnome-terminal --geometry=50x20+800+0 -e "AnT libhormo -m server & bash"
\# 这行命令会开启一个新的gnome-terminal终端窗口，设置其大小和位置（宽50个字符，高20行，距离屏幕左侧800像素，顶部0像素）。
\# '-e' 选项后面跟随的命令是在终端中执行的命令。这里执行'AnT libhormo -m server & bash'，
\# 'AnT' 似乎是一个程序，'libhormo' 是一个参数，'-m server' 指定了模式为server。
\# '&' 使得bash命令在后台执行，这允许在运行server模式的AnT命令后立即启动bash shell。

\# 接下来的几行都是类似的，它们在新的标签页中开启gnome-terminal终端窗口，并在每个终端中执行客户端模式的AnT命令。
gnome-terminal --tab -e "AnT libhormo -m client -n 10"
\# '-e' 选项指定终端执行'AnT libhormo -m client -n 10'，'-m client' 表明模式为client，'-n 10' 可能指定了一些并发数量或其他参数。

\# 后面的命令用 '&' 结尾，表示在后台并发执行这些命令，允许脚本继续执行而不等待每个命令完成。
gnome-terminal --tab -e "AnT libhormo -m client -n 10"&
gnome-terminal --tab -e "AnT libhormo -m client -n 10"&
gnome-terminal --tab -e "AnT libhormo -m client -n 10"&
gnome-terminal --tab -e "AnT libhormo -m client -n 10"&
gnome-terminal --tab -e "AnT libhormo -m client -n 10"
\# 每行命令都打开了一个新的终端标签页，并在该标签页中执行相同的客户端命令。

\# 这个脚本用于快速启动多个终端会话，以便在一个会话中运行服务器模式的程序，在其他会话中运行多个客户端实例。

