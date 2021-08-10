project(markable CXX)
cmake_minimum_required(VERSION 3.18)
enable_testing()

add_library(markable_lib INTERFACE)
target_include_directories(markable_lib INTERFACE $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>)
target_compile_features(markable_lib INTERFACE cxx_std_20)
target_compile_definitions(markable_lib INTERFACE AK_TOOLBOX_NO_UNDERLYING_TYPE)
add_library(markable::markable ALIAS markable_lib)

add_executable(test_markable test/test_markable.cpp)
target_link_libraries(test_markable PRIVATE markable_lib)
target_compile_options(test_markable PRIVATE -Wall -Wextra)
add_test(test_markable test_markable)
