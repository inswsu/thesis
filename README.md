# A model for studying the entrainment of oscillations in a pulse-modulated control system
> 关于研究脉冲调制控制系统中振荡夹带的模型

##3 libhormo.cpp
> 这段代码是一个用于定义动态系统映射的C++源文件，它可能是AnT系统的一部分。AnT系统看起来是一个仿真或数学建模工具，此代码定义了一个离散时间动态系统。下面是代码旁边的注释和说明：

#include "AnT.hpp"  // 包含AnT系统的头文件。

// 定义了几个宏以便于引用状态变量和参数。
#define X_DEF   currentState[0] // 状态变量x。
#define THETA_DEF   currentState[1] // 状态变量theta。

#define M_PI   3.14159265358979323846 // 定义了π的值。

// 参数宏定义，用于简化代码中参数的引用。
#define b   parameters[0]
#define k1  parameters[1]
// 省略中间部分参数的定义...
#define T   parameters[8]

#define omega   2*M_PI/T  // 计算角频率。
#define lambda   -b*T/(2*M_PI)  // 计算lambda参数。

// 动态系统的映射函数。
bool hormo_map (const Array<real_t>& currentState, const Array<real_t>& parameters, Array<real_t>& RHS){
	double z = M * (1.0 + cos(THETA_DEF)) + X_DEF;  // 计算z变量。
	double zhp = pow(z / h, p); // 计算z/h的p次幂。
	double func_F = k3 * z / (k4 + zhp); // 计算F函数。
	double func_Phi = k1 + k2 * zhp / (1.0 + zhp); // 计算Phi函数。
	RHS[0] = exp(lambda * func_Phi) * (X_DEF + func_F); // 计算新的x值。
	RHS[1] = fmod( THETA_DEF + omega * func_Phi, 2.0 * M_PI);// 计算新的theta值。
	return true; // 返回true表示映射成功。
}

// 外部"C"用于防止C++名称修饰，以便AnT系统能够在不知道C++编译细节的情况下调用这些函数。
extern "C" 
{
	// 系统连接函数。
	void connectSystem ()
	{
		MapProxy::systemFunction = hormo_map; // 将hormo_map函数注册为系统函数。
	}
}
##### 执行流程:
·从AnT系统中包含必要的头文件。
·定义了系统的当前状态和参数的宏，以便更方便地引用。
·实现了一个名为hormo_map的函数，这个函数根据当前状态和参数来计算动态系统的下一个状态。
·使用extern "C"，定义了一个connectSystem函数，该函数在加载时由AnT系统调用，以注册hormo_map函数作为系统映射函数。
·当这段代码编译成共享库并加载到AnT系统中时，connectSystem函数会被调用，注册hormo_map作为映射函数，允许AnT系统执行离散时间步进，以模拟和分析动态系统的行为。
##2 ./build-Ant-system.sh文件
> #### 目的
> (1)这个脚本是一个用于编译和创建共享库（在Windows中称为DLL，在Unix-like系统中称为so）的自动化脚本，特别是为了与AnT系统交互的C++代码。
> (2)这个脚本执行以下主要步骤：
·尝试识别并设置系统文件的名称。
·确认AnT命令的位置和所需的共享库扩展名。
·设置适合当前操作系统的编译器标志。
·编译C++源文件，生成对象文件。
·将对象文件链接为共享库（.dll或.so）。
·删除中间产物，打印共享库信息。
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

