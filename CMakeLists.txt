cmake_minimum_required (VERSION 2.8.11)

set (BLING_FIRE_VERSION_MAJOR 1)
set (BLING_FIRE_VERSION_MINOR 0)

project (BLING_FIRE)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/blingfireclient.library/inc)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/blingfireclient.library/src)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/blingfirecompile.library/inc)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/blingfirecompile.library/src)

IF (APPLE)
  set (OPTIMIZE "-O3")
ELSE()
  set (OPTIMIZE "-O4 -fexpensive-optimizations")
ENDIF()
set (CMAKE_CXX_FLAGS " -std=c++11 -fpic ${OPTIMIZE} -DNDEBUG")
set (CMAKE_CXX_FLAGS_RELEASE " -std=c++11 -fpic ${OPTIMIZE} -DNDEBUG")
set (CMAKE_CXX_FLAGS_DEBUG " -std=c++11 -fpic -O0 -g")
set (default_build_type "Release")

IF (UNIX OR LINUX)
  add_definitions ("-Wall -Wno-unknown-pragmas -DHAVE_ICONV_LIB -DHAVE_NO_SPECSTRINGS -D_VERBOSE -DBLING_FIRE_NOAP -DBLING_FIRE_NOWINDOWS")
  IF (APPLE)
    add_definitions (-DBLING_FIRE_MAC)
  ENDIF()
ELSE()
  add_definitions ("-DHAVE_NO_SPECSTRINGS -D_VERBOSE -DBLING_FIRE_NOAP")
ENDIF()


# define headers and sources
file(GLOB CLIENT_HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfireclient.library/inc/*.h")
file(GLOB CLIENT_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfireclient.library/src/*.cpp")
file(GLOB CLIENT_RESOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfireclient.library/src/*.cxx")

file(GLOB COMPILE_HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfirecompile.library/inc/*.h")
file(GLOB COMPILE_RESOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfirecompile.library/src/*.cxx")
file(GLOB COMPILE_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/blingfirecompile.library/src/*.cpp")

# build blingfire client
add_library(fsaClient STATIC ${CLIENT_HEADER_FILES} ${CLIENT_SOURCE_FILES} ${CLIENT_RESOURCE_FILES})

# build blingfire compile
add_library(fsaCompile STATIC ${CLIENT_HEADER_FILES} ${COMPILE_SOURCE_FILES} ${COMPILE_RESOURCE_FILES})
add_dependencies(fsaCompile fsaClient)

IF (UNIX OR LINUX)
  set_target_properties(fsaClient PROPERTIES LINKER_LANGUAGE CXX)
  set_target_properties(fsaCompile PROPERTIES LINKER_LANGUAGE CXX)
ENDIF()


# Macro for finding the sub directories
MACRO(SUBDIRLIST result curdir)
  FILE(GLOB children RELATIVE ${curdir} ${curdir}/*)
  SET(dirlist "")
  FOREACH(child ${children})
    IF(IS_DIRECTORY ${curdir}/${child})
      LIST(APPEND dirlist ${child})
    ENDIF()
  ENDFOREACH()
  SET(${result} ${dirlist})
ENDMACRO()

# find all tools dirs
SUBDIRLIST(SUBDIRS ${CMAKE_CURRENT_SOURCE_DIR}/blingfiretools)

# build tools
FOREACH(dir ${SUBDIRS})
  get_filename_component(dirname ${dir} NAME)
    file(GLOB sourcefile ${CMAKE_CURRENT_SOURCE_DIR}/blingfiretools/${dir}/*.cpp)
    file(GLOB resourcefile  ${CMAKE_CURRENT_SOURCE_DIR}/blingfiretools/${dir}/*.cxx)
    file(GLOB deffile ${CMAKE_CURRENT_SOURCE_DIR}/blingfiretools/${dir}/*.def)
    IF(${dirname} STREQUAL "blingfiretokdll")
      add_library(${dirname} SHARED ${sourcefile} ${resourcefile} ${deffile})
      target_link_libraries(${dirname} fsaClient fsaCompile)

      add_library(${dirname}_static ${sourcefile} ${resourcefile} ${deffile})
      target_link_libraries(${dirname}_static fsaClient fsaCompile)
    ELSE()
      add_executable(${dirname} ${sourcefile} ${resourcefile} ${deffile})
      target_link_libraries(${dirname} fsaCompile fsaClient)
      IF(${dirname} STREQUAL "any_test")
        target_link_libraries(${dirname} ${CMAKE_DL_LIBS})
      ENDIF()
      IF (APPLE)
        target_link_libraries(${dirname} iconv)
      ENDIF()
    ENDIF()

ENDFOREACH()

