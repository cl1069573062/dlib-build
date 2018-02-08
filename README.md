# Android NDK 编译 dlib.so
Android Studio下使用Cmake来编译dlib 动态库,以支持人脸识别和人脸68个特征点检测。仅编译人脸是被和检测特征点所需要的代码，编译后dlib.so 的大小为4.4M

### 环境准备:
Android Studio >= 2.2
dlib: v19.9
cmake:  V3.4.1

### 获取源码
官网左下角下载 http://dlib.net/

### 新建C 和 C++ 代码
##### 1 下载 NDK 和构建工具
1 在打开的项目中，从菜单栏选择 Tools > Android > SDK Manager。
2 点击 SDK Tools 标签。
3 选中 LLDB、CMake 和 NDK 旁的复选框

![attach_1510a5b62516f070](/image/1.png)

#### 创建支持 C/C++ 的新项目
1  在向导的 Configure your new project 部分，选中 Include C++ Support 复选框。
2 点击 Next。
3 正常填写所有其他字段并完成向导接下来的几个
部分。
4 在向导的 Customize C++ Support 部分，您可以使用下列选项自定义项目：
1. C++ Standard：使用下拉列表选择您希望使用哪种 C++ 标准。选择 Toolchain Default 会使用默认的 CMake 设置。
2. Exceptions Support：如果您希望启用对 C++ 异常处理的支持，请选中此复选框。如果启用此复选框，Android Studio 会将 -fexceptions 标志添加到模块级 build.gradle 文件的 cppFlags 中，Gradle 会将其传递到 CMake。
3. Runtime Type Information Support：如果您希望支持 RTTI，请选中此复选框。如果启用此复选框，Android Studio 会将 -frtti 标志添加到模块级 build.gradle 文件的 cppFlags 中，Gradle 会将其传递到 CMake。

#### 2 构建和运行示例应用
如果您想要验证 Gradle 是否已将原生库打包到 APK 中，可以使用 APK 分析器：

1 选择 Build > Analyze APK。
2 从 app/build/outputs/apk/ 目录中选择 APK 并点击 OK。
![attach_1510a5b62516f070](/image/2.png)您会在 APK 分析器窗口的 lib/<ABI>/ 下看到 libnative-lib.so。
![attach_1510a6131d186e14](/Users/chenlei/Documents/attach_1510a6131d186e14.png)

### 项目添加dlib源码
![attach_151141beda0daaec](/image/3.png)
### 修改CMakeLists.text
```cpp
cmake_minimum_required(VERSION 3.4.1)

set(CMAKE_VERBOSE_MAKEFILE on)

set(DLIB_DIR ${CMAKE_SOURCE_DIR}/src/main/dlib)
set(distribution_DIR ${CMAKE_SOURCE_DIR}/../distribution)

# Opencv and it will use static import
set(ANDROID_NDK_ABI_NAME ${CMAKE_ANDROID_ARCH_ABI})

# Include headers
include_directories(${DLIB_DIR} include)

add_library(dlib SHARED
            ${DLIB_DIR}//dlib/dnn/cpu_dlib.cpp
            ${DLIB_DIR}//dlib/dnn/tensor_tools.cpp
            ${DLIB_DIR}//dlib/threads/async.cpp
            ${DLIB_DIR}//dlib/threads/thread_pool_extension.cpp
            ${DLIB_DIR}//dlib/threads/threads_kernel_shared.cpp
            ${DLIB_DIR}/dlib/entropy_decoder/entropy_decoder_kernel_2.cpp
            ${DLIB_DIR}/dlib/base64/base64_kernel_1.cpp
            ${DLIB_DIR}/dlib/threads/threads_kernel_1.cpp
            ${DLIB_DIR}/dlib/threads/threads_kernel_2.cpp
           )

file(MAKE_DIRECTORY ${distribution_DIR})
file(MAKE_DIRECTORY ${distribution_DIR}/jniLibs)

set_target_properties(dlib PROPERTIES
                      LIBRARY_OUTPUT_DIRECTORY
                      ${distribution_DIR}/jniLibs/${ANDROID_ABI})


target_link_libraries(dlib)


```
`add_library `添加需要编译的cpp代码  `SHARED` 动态库
`set_target_properties ` 编译后的so 从 build/intermediates/cmake copy到distribution/jniLibs

### 修改build.gradle
```
  defaultConfig {
        externalNativeBuild {
            cmake {
                arguments "-DANDROID_STL=c++_shared"
                cppFlags "-std=c++11 -frtti -fexceptions"
            }
        }
    }
```
之后就开始编译
### 编译常见问题
1  error: 'to_string' is not a member of 'std'
在build.gradle 中cmake下添加     ` arguments "-DANDROID_STL=c++_shared"`
2 编译后的so一直为debug版本
debug版本的dlib动态库，人脸识别运行非常缓慢大概需要12秒，release版本大概为3秒（不同设备性能）
经测试一下来
1. 使用CmakeLists.txt指定，无效
2. 使用build.gradle的arguments指定-DBUILD_TYPE=Release，无效
3. cppFlags指定-O2或-O3，无效。

需要修改Android Studio Build Variant
![attach_1511461b445055fc](/image/4.png)
