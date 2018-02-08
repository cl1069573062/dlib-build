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

