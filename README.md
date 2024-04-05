# A model for studying the entrainment of oscillations in a pulse-modulated control system
> 关于研究脉冲调制控制系统中振荡夹带的模型

##2 ./build-Ant-system.sh文件
> #### 目的
> 这个脚本是一个用于编译和创建共享库（在Windows中称为DLL，在Unix-like系统中称为so）的自动化脚本，特别是为了与AnT系统交互的C++代码。
> #### 解释代码

\#!/bin/sh
\# 使用sh shell来执行此脚本。
\#
\# 请将此脚本复制到包含您的cpp文件（动态系统文件）的目录中，
\# 然后执行它以获取所需的系统dll。
\#

ANT_SYSTEM_NAME=libhormo  # 默认的系统名称。
USER_FLAGS=                # 初始化用户标志变量。

\# 遍历所有传递给脚本的参数。
for ARG in $@ ; do
  \# 检测ANT_SYSTEM_NAME是否已经被设置。
  if test "x${ANT_SYSTEM_NAME}" != "x" ; then
    USER_FLAGS="${USER_FLAGS} ${ARG}"  # 收集用户参数。
    continue
  fi

  \# 从参数中去掉'ANT_SYSTEM_NAME='前缀。
  ANT_SYSTEM_NAME=`echo "${ARG}" | sed 's/ANT_SYSTEM_NAME=//'`
  # 如果参数不包含前缀，则将它添加到USER_FLAGS。
  if test "x${ARG}" = "x${ANT_SYSTEM_NAME}" ; then
    ANT_SYSTEM_NAME=
    USER_FLAGS="${USER_FLAGS} ${ARG}"
  fi
done

\# 如果没有找到系统文件，则尝试自动探测。
if test "x${ANT_SYSTEM_NAME}" = "x" ; then
  \# 查找包含'AnT.h'或'connectSystem'的cpp文件。
  ANT_SYSTEM_FILE=`ls *.cpp | xargs -i egrep -l -e 'AnT.h' -e 'connectSystem' '{}'`

  \# 如果找不到系统文件，则打印错误并退出。
  if test "x${ANT_SYSTEM_FILE}" = "x" ; then
    echo
    echo " No system file found in the current directory!"
    echo
    exit 1
  fi

  # 从找到的文件列表中获取最后一个文件。
  for WORD in ${ANT_SYSTEM_FILE} ; do
  continue
  done
  \# 如果探测到多个文件，通知用户并退出。
  if test "${ANT_SYSTEM_FILE}" != "${WORD}" ; then
    echo
    echo " Ambiguous system file detection:"
    echo ${ANT_SYSTEM_FILE}
    echo
    echo " To disambiguate this just call:"
    echo
    echo "  `basename \"$0\"` ANT_SYSTEM_NAME=<file name without .cpp-suffix>"
    echo
    exit 1
  fi

  # 从系统文件名中去掉.cpp后缀以获取系统名称。
  ANT_SYSTEM_NAME=`basename ${ANT_SYSTEM_FILE} .cpp`;
fi

\# 检查ANT_TOPDIR环境变量是否被设置。
if $[ "x${ANT_TOPDIR}" = "x" ]$ ; then
  ANT_CMD=AnT  # 如果未设置，则默认命令为AnT。

  \# 如果找不到AnT命令，打印错误并退出。
  if $[ "x`which ${ANT_CMD}`" = "x" ]$ ; then
    echo "AnT not found!"
    exit 1
  fi
else
  # 如果设置了ANT_TOPDIR，则使用完整路径来定位AnT命令。
  ANT_CMD="${ANT_TOPDIR}"/bin/AnT
fi

\# 获取共享库的扩展名。
ANT_SHARED_LIB_EXT=`"${ANT_CMD}" --shared-lib-ext`

\# 下面的代码处理MSYS (MinGW)上的路径变量，以便处理空格或其他特殊字符。
\# 确定AnT安装前缀。
ANT_INSTALLATION_PREFIX=`cd "\`\"${ANT_CMD}\" --installation-prefix\`" && pwd`

\# 根据操作系统确定编译标志。
if $[ "x${ANT_SHARED_LIB_EXT}" = "xdll" ]$ ; then
   ANT_CXXFLAGS="-mno-cygwin -mms-bitfields"
   ANT_LDFLAGS="-shared -mno-cygwin"
   ANT_LIB_DIR="bin"
else \
   ANT_CXXFLAGS="-fPIC"
   ANT_LDFLAGS="-shared"
   ANT_LIB_DIR="lib"
fi

\# 编译.cpp文件，生成.o对象文件。
echo "Compiling ${ANT_SYSTEM_NAME}.cpp ..."
g++ -c ${USER_FLAGS} ${ANT_CXXFLAGS} ${ANT_SYSTEM_NAME}.cpp \
  -I"${ANT_INSTALLATION_PREFIX}"/include/AnT/engine \
  -o ${ANT_SYSTEM_NAME}.o \
|| exit

\# 构建共享库。
echo "Building shared library ${ANT_SYSTEM_NAME}.${ANT_SHARED_LIB_EXT} ..."
g++ ${USER_FLAGS} ${ANT_LDFLAGS} ${ANT_SYSTEM_NAME}.o \
  -L"${ANT_INSTALLATION_PREFIX}"/${ANT_LIB_DIR} -lAnT \
  -o ${ANT_SYSTEM_NAME}.${ANT_SHARED_LIB_EXT} \
|| exit

\# 删除临时生成的.o对象文件。
rm ${ANT_SYSTEM_NAME}.o

\# 打印出可用的AnT系统函数插件。
echo "Available AnT system function plugin: ${ANT_SYSTEM_NAME}.${ANT_SHARED_LIB_EXT}"

---------------------------------------------------------------------------------------------------------
##1 ./star.sh文件
> #### 目的：
> 在具体执行之前，需要了解AnT和libhormo是什么，因为这些参数对于理解代码的具体作用至关重要。如果它们是特定应用程序的组成部分，这个脚本可能用于开启一个服务器实例和多个客户端实例来进行通信或处理数据。
> #### 解释代码：
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

