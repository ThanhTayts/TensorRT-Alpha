cmake_minimum_required(VERSION 3.5)

set(CMAKE_CUDA_ARCHITECTURES 60 61 62 70 72 75)
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)

project(detection LANGUAGES CXX CUDA)
set(CMAKE_BUILD_TYPE Debug)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -O3 -g")
set(CMAKE_CXX_STANDARD 14)
option(CUDA_USE_STATIC_CUDA_RUNTIME OFF)

# CUDA
find_package(CUDA REQUIRED)
include_directories(/usr/local/cuda/include)
link_directories(/usr/local/cuda/lib64)
message(STATUS "CUDA Libs: \n${CUDA_LIBRARIES}\n")
message(STATUS "CUDA Headers: \n${CUDA_INCLUDE_DIRS}\n")
include_directories(${CUDA_INCLUDE_DIRS} /usr/local/cuda/include/nvtx3)
# Include cuBLAS headers
include_directories(/usr/local/cuda-10.2/targets/aarch64-linux/include)
# Link cuBLAS library
find_library(CUBLAS_LIB cublas HINTS /usr/local/cuda-10.2/targets/aarch64-linux/lib)
if (NOT CUBLAS_LIB)
    message(FATAL_ERROR "cuBLAS library not found!")
endif()

# OpenCV
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV Libs: \n${OpenCV_LIBS}\n")
message(STATUS "OpenCV Libraries: \n${OpenCV_LIBRARIES}\n")
message(STATUS "OpenCV Headers: \n${OpenCV_INCLUDE_DIRS}\n")

# TensorRT
set(TensorRT_INCLUDE_DIRS /usr/include/x86_64-linux-gnu)
set(TensorRT_LIBRARIES /usr/lib/x86_64-linux-gnu)

message(STATUS "TensorRT Libs: \n${TensorRT_LIBRARIES}\n")
message(STATUS "TensorRT Headers: \n${TensorRT_INCLUDE_DIRS}\n")

get_filename_component(LIBS_COMMON_TENSORRT_DIR ../../common/libs/inc/ ABSOLUTE)
message("LIBS_COMMON_TENSORRT_DIR: ${LIBS_COMMON_TENSORRT_DIR}")

if(EXISTS ${LIBS_COMMON_TENSORRT_DIR})
    message(STATUS "LIBS_COMMON_TENSORRT_DIR exists: ${LIBS_COMMON_TENSORRT_DIR}")
    set(INC_DIR_TO_USE "${LIBS_COMMON_TENSORRT_DIR}")
    add_definitions(-DLIB_COMMON=1)
    add_definitions(-DJETSON_PLATFORM=1)
else()
    message(STATUS "LIBS_COMMON_TENSORRT_DIR does not exist, skipping")
    set(INC_DIR_TO_USE "${CMAKE_CURRENT_SOURCE_DIR}/common")
    add_definitions(-DLIB_COMMON=0)
endif()

list(APPEND INCLUDE_DIRS
        ${CUDA_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
        ${TensorRT_INCLUDE_DIRS}
        ${INC_DIR_TO_USE}
        )

list(APPEND ALL_LIBS
        ${CUDA_LIBRARIES}
        ${OpenCV_LIBRARIES}
        ${TensorRT_LIBRARIES}
        ${CUBLAS_LIB}
        )
        
set(SOURCE
        common/logger.cpp
        src/kernel_function.cu
        src/utils.cpp
        src/yolo.cpp
        )

add_library(${PROJECT_NAME} STATIC ${SOURCE})

target_include_directories(${PROJECT_NAME} PUBLIC inc common ${INCLUDE_DIRS})

target_link_libraries(${PROJECT_NAME} 
                        PUBLIC 
                        nvinfer 
                        nvinfer_plugin 
                        cudart 
                        nvToolsExt 
                        dl
                        ${OpenCV_LIBS}
                        ${CUBLAS_LIB}
                        nppc                     
                        nppial                   
                        nppicc                   
                        nppidei                  
                        nppif        
                        nppig                  
                        nppim                   
                        nppist                  
                        nppisu                   
                        )