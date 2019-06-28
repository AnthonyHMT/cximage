cmake_minimum_required(VERSION 3.13)
project(cximage)

if (NOT WIN32 AND NOT UNIX OR APPLE)
    message(FATAL_ERROR "Unsupported Platform!")
endif()

set(CMAKE_CXX_STANDARD 11)

include_directories(
        ${CMAKE_SYSROOT}/include
        ${CMAKE_SYSROOT}/usr/include
)

link_directories(
        ${CMAKE_SYSROOT}/lib
        ${CMAKE_SYSROOT}/usr/lib
)

file(GLOB_RECURSE SRC_FILES "src/*.h" "src/*.cpp")

add_library(cximage SHARED ${SRC_FILES})
set_target_properties(cximage PROPERTIES VERSION 1.0 SOVERSION 1)

option(CXIMG_WITH_JPG "Build with jpeg supported!" ON)
option(CXIMG_WITH_PNG "Build with png supported!" ON)
option(CXIMG_WITH_TIF "Build with tiff supported!" ON)
option(CXIMG_WITH_JASPER "Build with jasper supported!" ON)
option(CXIMG_WITH_MNG "Build with mng supported!" OFF)

if (CXIMG_WITH_JPG)
    add_definitions(-DSUPPORT_JPG)
    list(APPEND EXTRA_LIBS libjpeg)
    list(APPEND EXTRA_PKGS libjpeg)
endif()
if (CXIMG_WITH_PNG)
    add_definitions(-DSUPPORT_PNG)
    list(APPEND EXTRA_LIBS libpng)
    list(APPEND EXTRA_PKGS libpng)
endif()
if (CXIMG_WITH_TIF)
    add_definitions(-DSUPPORT_TIFF)
    list(APPEND EXTRA_LIBS libtiff)
    list(APPEND EXTRA_PKGS libtiff-4)
endif()
if (CXIMG_WITH_MNG)
    add_definitions(-DSUPPORT_MNG)
    list(APPEND EXTRA_LIBS libmng)
    list(APPEND EXTRA_PKGS libmng)
endif()
if (CXIMG_WITH_JASPER)
    add_definitions(-DWITH_JASPER)
    list(APPEND EXTRA_LIBS libjasper)
    list(APPEND EXTRA_PKGS jasper)
endif()

list(APPEND EXTRA_LIBS libzlib)
list(APPEND EXTRA_PKGS zlib)

if(WIN32)
    set_target_properties(cximage PROPERTIES PREFIX "")
    target_link_libraries(cximage ${EXTRA_LIBS})
elseif(UNIX AND NOT APPLE)
    add_definitions(-D_LINUX)

    if(DEFINED CMAKE_SYSROOT)
        set(PKG_CONFIG_USE_CMAKE_PREFIX_PATH ${CMAKE_SYSROOT}/usr/lib/pkgconfig)
        set(PKG_CONFIG_PATH ${CMAKE_SYSROOT}/usr/lib/pkgconfig)
        set(PKG_CONFIG_EXECUTABLE ${CMAKE_SYSROOT}/../../bin/pkg-config)
    endif()

    #find_package(jasper)
    find_package(PkgConfig REQUIRED)

    pkg_check_modules(REQLIBS REQUIRED ${EXTRA_PKGS})

    #message("${REQLIBS_INCLUDE_DIRS}  || ${REQLIBS_LIBRARIES}")

    include_directories(${REQLIBS_INCLUDE_DIRS})
    target_link_libraries(cximage ${REQLIBS_LIBRARIES})
endif()

list(APPEND CXIMAGE_HEADER
        src/ximage.h src/xfile.h src/xiofile.h src/xmemfile.h
        src/ximacfg.h src/ximadef.h)

set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")

install(TARGETS cximage
        RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}")

install(FILES ${CXIMAGE_HEADER} DESTINATION include/cximage)