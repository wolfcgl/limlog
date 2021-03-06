cmake_minimum_required(VERSION 2.9)
project(limlog C CXX)

# set(CMAKE_VERBOSE_MAKEFILE ON)
option(LIMLOG_CLANG "clang++" OFF)
option(LIMLOG_NO_FILE_FUNC_LINE "not show file-func-line" OFF)

if(WIN32)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS)
else()
	set(CMAKE_CXX_COMPILER "g++")
	set(CMAKE_CXX_FLAGS "-std=c++11 -O3 -Wall -Werror")
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__REL_FILE__='\"$(subst ${PROJECT_SOURCE_DIR}/,,$(abspath $<))\"'")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread")

if (LIMLOG_CLANG)
	set(CMAKE_CXX_COMPILER "clang++")
endif()

if (LIMLOG_NO_FILE_FUNC_LINE)
	add_definitions(-DNO_FILE_FUNC_LINE)
endif()

add_subdirectory(lim)
add_subdirectory(tests)