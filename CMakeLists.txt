project(markable)
cmake_minimum_required(VERSION 2.6)
enable_testing()

set(CMAKE_CXX_FLAGS "-std=c++0x -Wall -Wextra -DAK_TOOLBOX_NO_UNDERLYING_TYPE")

add_executable(test_markable test/test_markable.cpp)

add_test(test_markable test_markable)
