# Copyright (C) 2019 ~ 2020 Uniontech Software Technology Co.,Ltd.
# SPDX-FileCopyrightText: 2022 UnionTech Software Technology Co., Ltd.
#
# SPDX-License-Identifier: GPL-3.0-or-later

cmake_minimum_required(VERSION 3.7)

if (NOT DEFINED VERSION)
    set(VERSION 5.5.10.1)
endif ()

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option (PERF_ON "Use provided math implementation" ON)

# 设置包含头文件的时候不用包含路径 begin ****************************************************************************************
macro(SUBDIRLIST result curdir)
    file(GLOB children RELATIVE ${curdir} ${curdir}/*)
    set(dirlist "")
    foreach(child ${children})
        if(IS_DIRECTORY ${curdir}/${child})
            LIST(APPEND dirlist ${child})
        endif()
    endforeach()
    set(${result} ${dirlist})
endmacro()

# 引用ZeroMQ的库 begin ****************************************************************
link_libraries("zmq")
# 引用ZeroMQ的库 end  *****************************************************************

find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

find_package(PkgConfig REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5 COMPONENTS DBus REQUIRED)
find_package(Qt5Gui REQUIRED)
find_package(Qt5Sql REQUIRED)
find_package(DtkWidget REQUIRED)
find_package(Qt5Network REQUIRED)
find_package(DtkCore REQUIRED)
find_package(PolkitQt5-1 REQUIRED)
find_package(QApt REQUIRED)

#PKG_SEARCH_MODULE(kmod REQUIRED libkmod IMPORTED_TARGET)

set(PROJECT_NAME_TEST
    ${PROJECT_NAME}-test)

ADD_COMPILE_OPTIONS(-fno-access-control)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../src)
foreach(subdir ${all_src})
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/../src/${subdir})
endforeach()

#src
file(GLOB_RECURSE APP_SRCS
     ${CMAKE_CURRENT_LIST_DIR}/../src/*.cpp
    )
# remove src main.cpp or will multi define
list(REMOVE_ITEM APP_SRCS ${CMAKE_CURRENT_LIST_DIR}/../src/main.cpp)

#test src
file(GLOB_RECURSE TEST_SRC_CPP ${CMAKE_CURRENT_LIST_DIR}/src/*.cpp)
file(GLOB_RECURSE TEST_SRC_H ${CMAKE_CURRENT_LIST_DIR}/src/*.h)

add_executable(${PROJECT_NAME_TEST} ${APP_SRCS} ${TEST_SRC_CPP} ${TEST_SRC_H})

target_include_directories(${PROJECT_NAME_TEST}
    PUBLIC ${DtkWidget_INCLUDE_DIRS} ${OBJECT_BINARY_DIR})

target_link_libraries(${PROJECT_NAME_TEST}
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
    Qt5::DBus
    Qt5::Sql
    Qt5::Network
    ${DtkWidget_LIBRARIES}
    ${DtkCore_LIBRARIES}
    ${DtkGUI_LIBRARIES}
    ${GTEST_LIBRARIES}
    ${GTEST_MAIN_LIBRARIES}
    PolkitQt5-1::Agent
    pthread
    kmod
    QApt
    )

add_custom_target(test-server
    COMMAND echo " =================== TEST BEGIN ==================== "
    COMMAND ${CMAKE_BINARY_DIR}/deepin-devicemanager-server/tests/${PROJECT_NAME_TEST}
    COMMAND echo " =================== TEST END ==================== "
)
#'make test'命令依赖与我们的测试程序
add_dependencies(test ${PROJECT_NAME_TEST})

# 设置添加gocv相关信息的输出
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -fprofile-arcs -ftest-coverage -fstack-protector-strong -D_FORTITY_SOURCE=1 -z noexecstack -pie -fPIC -z lazy")

if(CMAKE_SAFETYTEST STREQUAL "CMAKE_SAFETYTEST_ARG_ON")
  #安全测试选项
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -fsanitize=undefined,address -O2")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -fsanitize=undefined,address -O2")
endif()
