cmake_minimum_required(VERSION 3.0)
project(yolov5_list)

set(CMAKE_CXX_STANDARD 11)

# 指定 ncnn 的安装路径
set(ncnn_DIR ncnn/lib/cmake/ncnn)

# 查找 ncnn
find_package(ncnn REQUIRED)

find_package(OpenCV REQUIRED)

# 输出 OpenCV 版本
message(STATUS "OpenCV version: ${OpenCV_VERSION}")



# 添加可执行文件,yolov5_lite是最终的执行文件的名字
add_executable(yolov5_lite yolov5_lite_e.cpp)

# 链接到 ncnn
target_link_libraries(yolov5_lite ncnn)
# 链接到 OpenCV，这里必须提前安装了opencv
target_link_libraries(yolov5_lite ${OpenCV_LIBS})
# 包含 OpenCV 头文件目录
target_include_directories(yolov5_lite PRIVATE ${OpenCV_INCLUDE_DIRS})


