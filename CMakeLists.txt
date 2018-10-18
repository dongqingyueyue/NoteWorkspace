cmake_minimum_required(VERSION 2.8)
project(Myeigen)

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)
find_package(Eigen3 3.1.0 REQUIRED)


include_directories(
${EIGEN3_INCLUDE_DIR}
)

#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED

first.cpp
)
add_executable(first first.cpp)


