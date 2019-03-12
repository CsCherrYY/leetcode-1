# Cmake tutoriol

本教程运行环境为ubuntu 16.04，cmake 版本3.5.1

安装cmake：

`apt install cmake`

## Step1 起步

最基础的项目工程是通过单一源文件编译得到可执行文件。对于这样简单的工程，只需要两行CMakeLists.txt代码即可实现。以下是一个简单示例。

先建立一个文件夹，用于存放我们的工程。

```bash
mkdir -p cmake/step1
cd cmake/step1
touch tutorial.cxx
touch CMakeLists.txt
```

源代码tutoriol.cxx实现了求一个数的平方根的功能：

```cpp
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <string>

int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }
  double inputValue = atof(argv[1]);

  double outputValue = sqrt(inputValue);
  std::cout << "The square root of " << inputValue 
            << " is " << outputValue << std::endl;
  return 0;
}

```

CMakeLists.txt的内容：

```bash
project (Tutorial)
add_executable(Tutorial tutorial.cxx)
```

CMakeLists.txt中的命令支持大写、小写，大小写混合。本例中使用小写。

编译的时候，在当前目录下新建一个build目录，用于存放编译过程产生的文件以及生成的可执行文件。

```bash
mkdir build
cd build
cmake ..
make
```

cmake .. 表示在..（上级）目录中寻找CMakeLists.txt进行编译，cmake执行成功后会自动生成Makefile文件，执行make即可完成编译。

cmake 执行结果：

```text
root@x:~/qinrui/cmake/step1/build# cmake ..
-- The C compiler identification is GNU 5.4.0
-- The CXX compiler identification is GNU 5.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /root/qinrui/cmake/step1/build
```

make执行结果：

```bash
root@x:~/qinrui/cmake/step1/build# make
Scanning dependencies of target Tutorial
[ 50%] Building CXX object CMakeFiles/Tutorial.dir/tutorial.cxx.o
[100%] Linking CXX executable Tutorial
[100%] Built target Tutorial
```

### 添加版本号和配置头文件

我们要给当前工程添加的第一个特性是版本号信息。尽管可以在源代码中完成该操作，但在CMakeLists.txt中完成此操作能够大大提升灵活性。我们可以按照以下方式修改CMakeLists.txt

注意，在CMakeLists.txt中，"\#"表示注释。

```text
cmake_minimum_required (VERSION 3.3)
project (Tutorial)
# 版本号信息
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)
 
# 配置一个头文件，从而将CMake的一些设置传递到源文件中。
# 以TutorialConfig.h.in为模板，生成TutorialConfig.h
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )
 
# 将构建目录添加到include的搜索路径中
# 这样就能够找到TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")
 
# 添加可执行文件名
add_executable(Tutorial tutorial.cxx)
```

因为配置文件会被写入构建目录中，所以我们将这个目录添加到include文件的搜索范围中。

我们在工程中添加头文件模板TutorialConfig.h.in：

```text
// 配置信息
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

当Cmake生成这个配置文件时，@Tutorial\_VERSION\_MAJOR@ 和 @Tutorial\_VERSION\_MINOR@ 将会被CMakeLists.txt中设置的值替换。接下来我们修改tutorial.cxx来include头文件以获取版本号。

添加了一条include以引用TutorialConfig.h。在usage 信息中添加了版本号。

```cpp
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <string>
#include "TutorialConfig.h"
int main(int argc, char *argv[])
{
  if (argc < 2)
  {
    std::cout << argv[0] << " Version "
              << Tutorial_VERSION_MAJOR << "."
              << Tutorial_VERSION_MINOR << std::endl;
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  double inputValue = atof(argv[1]);

  double outputValue = sqrt(inputValue);
  std::cout << "The square root of " << inputValue
            << " is " << outputValue << std::endl;
  return 0;
}

```

将build的内容清空，重新cmake，make编译。

```bash
cd build
rm -rf *
cmake ..
make
```

此时的文件目录结构为：（省略了CMakeFiles内的文件）

```bash
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   ├── Makefile
│   ├── Tutorial
│   └── TutorialConfig.h
├── CMakeLists.txt
├── TutorialConfig.h.in
└── tutorial.cxx
```

运行结果：

```bash
root@x:~/qinrui/cmake/step1/build# ./Tutorial 
./Tutorial Version 1.0
Usage: ./Tutorial number
```

## Step2 添加库文件

接下来我们将会往项目中添加库文件。这个库包含了我们自己写的sqrt的实现，以替代系统库。本教程中我们将库文件放入MathFunctions子目录中。这个字母录也包含一个CMakeLists.txt文件，只有一行

```text
add_library(MathFunctions mysqrt.cxx)
```

添加MathFunctions.h头文件，同样只有一行定义：

```cpp
double mysqrt(double x);
```

在mysqrt.cxx中给出mysqrt实现：

```cpp
#include "MathFunctions.h"
#include <iostream>

double mysqrt(double x)
{
	if (x <= 0)
	{
		return 0;
	}

	double result = x;

	// 循环计算10次
	for (int i = 0; i < 10; ++i)
	{
		if (result <= 0)
		{
			result = 0.1;
		}
		double delta = x - (result * result);
		result = result + 0.5 * delta / result;
	}
	return result;
}

```

在mysqrt.cxx源代码中提供了mysqrt函数，实现了和sqrt函数相同的功能。为了使用新的库，我们在顶层CMakeLists中添加一行add\_subdirectory调用，这样库文件就会被构建。同时添加include路径，使得MathFunctions/MathFunctions.h 头文件中定义的函数原型能被找到。最后一步是将库文件添加到可执行文件中。在顶层CMakeLists.txt中添加：

```text
include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
add_subdirectory (MathFunctions) 
 
# 添加可执行文件
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial MathFunctions)
```

下面考虑将MathFunctions这个库设置成可选的。当使用较大的库或者第三方库时会用到该功能。在顶层CMakeLists.txt中添加：

```text
# 是否使用自定义函数库
option (USE_MYMATH "Use tutorial provided math implementation" ON) 
```

下一步我们要将MathFunctions库的构建和链接设置成可选的。对此我们做出如下修改：

```text
# 是添加自定义库
if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
 
# 添加可执行文件
add_executable (Tutorial tutorial.cxx)
target_link_libraries (Tutorial  ${EXTRA_LIBS})
```

使用USE\_MYMATH来判定是否需要编译使用MathFunctions。这里使用了一个EXTRA\_LIBS变量来收集所有可选的库，用于链接到可执行文件。这是一种保证具有多重选择的大型项目整洁性的常用方法。相关源代码的修改如下：

```cpp
#include <cmath>
#include <iostream>
#include <string>

#include "TutorialConfig.h"

#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif

int main(int argc, char *argv[])
{
	if (argc < 2)
	{
		std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR
				  << "." << Tutorial_VERSION_MINOR << std::endl;
		std::cout << "Usage: " << argv[0] << " number" << std::endl;
		return 1;
	}

	double inputValue = std::stod(argv[1]);

#ifdef USE_MYMATH
	double outputValue = mysqrt(inputValue);
	std::cout << "Mysqrt: The square root of " << inputValue
			  << " is " << outputValue << std::endl;
#else
	double outputValue = sqrt(inputValue);
	std::cout << "sqrt:  The square root of " << inputValue
			  << " is " << outputValue << std::endl;
#endif

	return 0;
}

```

在源代码中我们同样使用了USE\_MYMATH，CMake通过TutorialConfig.h.in将该值传递到源代码中。需要在该文件内添加：

```bash
#cmakedefine USE_MYMATH
```

## Step3 添加库的依赖项

使用依赖项可以更好地控制库/可执行文件的链接和引用。同时也提供了对CMake内目标属性的控制。使用依赖项的主要语法是：

```text
  -  target_compile_definitions
  -  target_compile_options
  -  target_include_directories
  -  target_link_libraries
```

首先是MathFunctions。我们首先声明任何链接到MathFunctions的文件都 需要包含当前的源目录，而MathFunctions本身 没有。因此，这可以使用INTERFACE 依赖项。在MathFunction目录下的CMakeLists.txt中添加：

```text
# 声明所有链接到我们的都需要引用当前目录，
# 去寻找MathFunctions.h，而我们自身不需要
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
```

这样我们就为MathFunctions指定了使用要求，我们可以安全地删除之前使用的EXTRA\_INCLUDES变量。

## Step4 安装与测试

现在我们可以开始添加测试功能和安装规则。

安装规则相对简单：对MathFunctions我们安装库文件和头文件，对应用我们安装可执行文件并配置头文件。

在MathFunctions/CMakeLists.txt中添加：

```bash
install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

在顶层CMakeLists.txt中添加：

```bash
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
          DESTINATION include
          )
```

在经过cmake和make后，使用make installl 命令进行安装，输出如下：

```bash
root@x:~/qinrui/cmake/step4/build# make install 
[ 50%] Built target MathFunctions
[100%] Built target Tutorial
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/Tutorial
-- Installing: /usr/local/include/TutorialConfig.h
-- Installing: /usr/local/bin/libMathFunctions.a
-- Installing: /usr/local/include/MathFunctions.h
```

此时Tutorial即已经被安装到系统中，不需要再当前目录下执行，也不需要使用./运行。

接下来对我们的应用进行测试。在顶层CMakeLists.txt中我们可以添加一些测试样例来验证应用的正确性。

```text
# 启用测试
enable_testing()

# 测试应用能否运行
add_test(NAME Runs COMMAND Tutorial 25)

# 测试Usage语句是否正确
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# 定义一个测试样例函数
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# 添加测试样例
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

编译完成后，要运行这些测试，需要执行

```bash
ctest -N
ctest -VV
```

```bash
root@x:~/qinrui/cmake/step4/build# ctest -N
Test project /root/qinrui/cmake/step4/build
  Test #1: Runs
  Test #2: Usage
  Test #3: Comp25
  Test #4: Comp-25
  Test #5: Comp0.0001

Total Tests: 5
```

```bash
root@x:~/qinrui/cmake/step4/build# ctest -VV
...from :/root/qinrui/cmake/step4/build/DartConfiguration.tcl
...from :/root/qinrui/cmake/step4/build/DartConfiguration.tcl
Test project /root/qinrui/cmake/step4/build
Constructing a list of tests
Done constructing a list of tests
Checking test dependency graph...
Checking test dependency graph end
test 1
    Start 1: Runs

1: Test command: /root/qinrui/cmake/step4/build/Tutorial "25"
1: Test timeout computed to be: 9.99988e+06
1: Mysqrt: The square root of 25 is 5
1/5 Test #1: Runs .............................   Passed    0.00 sec
test 2
    Start 2: Usage

2: Test command: /root/qinrui/cmake/step4/build/Tutorial
2: Test timeout computed to be: 9.99988e+06
2: /root/qinrui/cmake/step4/build/Tutorial Version 1.0
2: Usage: /root/qinrui/cmake/step4/build/Tutorial number
2/5 Test #2: Usage ............................   Passed    0.00 sec
test 3
    Start 3: Comp25

3: Test command: /root/qinrui/cmake/step4/build/Tutorial "25"
3: Test timeout computed to be: 9.99988e+06
3: Mysqrt: The square root of 25 is 5
3/5 Test #3: Comp25 ...........................   Passed    0.00 sec
test 4
    Start 4: Comp-25

4: Test command: /root/qinrui/cmake/step4/build/Tutorial "-25"
4: Test timeout computed to be: 9.99988e+06
4: Mysqrt: The square root of -25 is 0
4/5 Test #4: Comp-25 ..........................   Passed    0.00 sec
test 5
    Start 5: Comp0.0001

5: Test command: /root/qinrui/cmake/step4/build/Tutorial "0.0001"
5: Test timeout computed to be: 9.99988e+06
5: Mysqrt: The square root of 0.0001 is 0.01
5/5 Test #5: Comp0.0001 .......................   Passed    0.00 sec

100% tests passed, 0 tests failed out of 5

Total Test time (real) =   0.01 sec
```























