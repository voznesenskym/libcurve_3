################################################################################
#  THIS FILE IS 100% GENERATED BY ZPROJECT; DO NOT EDIT EXCEPT EXPERIMENTALLY  #
#  Please refer to the README for information about making permanent changes.  #
################################################################################

########################################################################
# Project setup
########################################################################
cmake_minimum_required(VERSION 2.8)
project(curve)
enable_language(C)
enable_testing()

set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})

########################################################################
# determine version
########################################################################
foreach(which MAJOR MINOR PATCH)
    file(STRINGS "${SOURCE_DIR}/include/curve_library.h" CURVE_VERSION_STRING REGEX "#define CURVE_VERSION_${which}")
    string(REGEX MATCH "#define CURVE_VERSION_${which} ([0-9_]+)" CURVE_REGEX_MATCH "${CURVE_VERSION_STRING}")
    if (NOT CURVE_REGEX_MATCH)
        message(FATAL_ERROR "failed to parse CURVE_VERSION_${which} from curve.h")
    endif()
    set(CURVE_${which}_VERSION ${CMAKE_MATCH_1})
endforeach(which)

set(CURVE_VERSION ${CURVE_MAJOR_VERSION}.${CURVE_MINOR_VERSION}.${CURVE_PATCH_VERSION})

########################################################################
# platform.h
########################################################################
include(CheckIncludeFile)
CHECK_INCLUDE_FILE("linux/wireless.h" HAVE_LINUX_WIRELESS_H)
CHECK_INCLUDE_FILE("net/if_media.h" HAVE_NET_IF_MEDIA_H)

include(CheckFunctionExists)
CHECK_FUNCTION_EXISTS("getifaddrs" HAVE_GETIFADDRS)
CHECK_FUNCTION_EXISTS("freeifaddrs" HAVE_FREEIFADDRS)

include(CheckIncludeFiles)
check_include_files("sys/socket.h;net/if.h" HAVE_NET_IF_H)
if (NOT HAVE_NET_IF_H)
    CHECK_INCLUDE_FILE("net/if.h" HAVE_NET_IF_H)
endif()

file(WRITE ${BINARY_DIR}/platform.h.in "
#cmakedefine HAVE_LINUX_WIRELESS_H
#cmakedefine HAVE_NET_IF_H
#cmakedefine HAVE_NET_IF_MEDIA_H
#cmakedefine HAVE_GETIFADDRS
#cmakedefine HAVE_FREEIFADDRS
")

configure_file(${BINARY_DIR}/platform.h.in ${BINARY_DIR}/platform.h)

#The MSVC C compiler is too out of date,
#so the sources have to be compiled as c++
if (MSVC)
    enable_language(CXX)
    file(GLOB sources ${SOURCE_DIR}/src/*.c)
    set_source_files_properties(${sources} PROPERTIES LANGUAGE CXX)
    set(MORE_LIBRARIES ws2_32 Rpcrt4 Iphlpapi)
endif()

# required libraries for mingw
if (MINGW)
    set(MORE_LIBRARIES -lws2_32 -lrpcrt4 -liphlpapi)
endif()


list(APPEND CMAKE_MODULE_PATH ${SOURCE_DIR})

########################################################################
# ZMQ dependency
########################################################################
find_package(ZeroMQ REQUIRED)
include_directories(${ZEROMQ_INCLUDE_DIRS})
list(APPEND MORE_LIBRARIES ${ZEROMQ_LIBRARIES})

########################################################################
# CZMQ dependency
########################################################################
find_package(CZMQ REQUIRED)
include_directories(${CZMQ_INCLUDE_DIRS})
list(APPEND MORE_LIBRARIES ${CZMQ_LIBRARIES})

########################################################################
# SODIUM dependency
########################################################################
find_package(Sodium REQUIRED)
include_directories(${SODIUM_INCLUDE_DIRS})
list(APPEND MORE_LIBRARIES ${SODIUM_LIBRARIES})

########################################################################
# includes
########################################################################
set (curve_headers
    include/curve_library.h
    include/curve.h
    include/curve_client.h
    include/curve_server.h
    include/curve_codec.h
    include/curve_z85.h
)
source_group ("Header Files" FILES ${curve_headers})
install(FILES ${curve_headers} DESTINATION include)

########################################################################
# library
########################################################################
include_directories(${BINARY_DIR})
include_directories(${SOURCE_DIR}/include)
set (curve_sources
    src/curve_client.c
    src/curve_server.c
    src/curve_codec.c
    src/curve_z85.c
)
source_group ("Source Files" FILES ${curve_sources})
add_library(curve SHARED ${curve_sources})
set_target_properties(curve PROPERTIES DEFINE_SYMBOL "LIBCURVE_EXPORTS")
target_link_libraries(curve ${ZEROMQ_LIBRARIES} ${MORE_LIBRARIES})

install(TARGETS curve
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# pkgconfig
########################################################################
set(VERSION "${CURVE_VERSION}")
set(prefix "${CMAKE_INSTALL_PREFIX}")
set(exec_prefix "\${prefix}")
set(libdir "\${prefix}/lib${LIB_SUFFIX}")
set(includedir "\${prefix}/include")
configure_file(
    ${SOURCE_DIR}/src/libcurve.pc.in
    ${BINARY_DIR}/libcurve.pc
@ONLY)

install(
    FILES ${BINARY_DIR}/libcurve.pc
    DESTINATION lib${LIB_SUFFIX}/pkgconfig
)

########################################################################
# tests
########################################################################
add_executable(curve_selftest ${SOURCE_DIR}/src/curve_selftest.c)
target_link_libraries(curve_selftest curve ${ZEROMQ_LIBRARIES})
add_test(curve_selftest curve_selftest)

########################################################################
# summary
########################################################################
message(STATUS "version: ${CURVE_VERSION}")
message(STATUS "install: ${CMAKE_INSTALL_PREFIX}")

################################################################################
#  THIS FILE IS 100% GENERATED BY ZPROJECT; DO NOT EDIT EXCEPT EXPERIMENTALLY  #
#  Please refer to the README for information about making permanent changes.  #
################################################################################
