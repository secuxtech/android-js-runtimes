cmake_minimum_required(VERSION 3.6.0)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -DANDROID_STL=c++_static -DANDROID_TOOLCHAIN=clang -DDEBUGGER_SERVER_PORT=${DEBUGGER_SERVER_PORT}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DQUICKJS")
include_directories(
    src/main/c/quickjs
)

# 確保版本號與檔案內容相符（升級至 2025-09-13）
add_definitions(-DCONFIG_VERSION="2025-09-13")

add_library( fastdev_quickjs_runtime

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/c/quickjs_runtime.cpp

            src/main/c/quickjs/cutils.c
            src/main/c/quickjs/dtoa.c
            src/main/c/quickjs/libregexp.c
            src/main/c/quickjs/libunicode.c
            src/main/c/quickjs/quickjs.c
    )

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DCONFIG_VERSION=\\\"2025-09-13\\\"")

find_library(log-lib log)
target_link_libraries(fastdev_quickjs_runtime ${log-lib})

# Fix for NDK <22: Set page size for 16KB page alignment compatibility
# https://developer.android.com/guide/practices/page-sizes#compile-r22-lower
target_link_options(fastdev_quickjs_runtime PRIVATE "-Wl,-z,max-page-size=16384")
target_link_options(fastdev_quickjs_runtime PRIVATE "-Wl,-z,common-page-size=16384")

if(MSVC)
    # https://github.com/ekibun/flutter_qjs/issues/7
    target_compile_options(fastdev_quickjs_runtime PRIVATE "/Oi-")
endif()