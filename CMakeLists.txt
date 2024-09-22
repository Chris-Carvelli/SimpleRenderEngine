cmake_minimum_required(VERSION 3.11)
project(SimpleRenderEngine CXX)

if ("${CMAKE_CURRENT_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
    message(WARNING
            "Building from within the SimpleRenderEngine source tree is not recommended.\n"
            "Hint: Run these commands\n"
            "$ rm -rf CMakeCache.txt CMakeFiles\n"
            "$ mkdir -p ../sre_build\n"
            "$ cd ../sre_build\n"
            "$ cmake ${CMAKE_CURRENT_SOURCE_DIR}")
endif ()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# where to look for cmake "find" modules
set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# custom, centralized path to dependencies
set(SRE_LIBS_PATH "" CACHE PATH "Path to the SimpleRenderEngine libraries")

find_package(SDL2 REQUIRED)
set(SRE_INCLUDE_DIRS ${SDL2_INCLUDE_DIR})

#########################################################
# FIND OPENGL
#########################################################
IF(APPLE)
    add_definitions(-DGL_SILENCE_DEPRECATION)
    list(APPEND SRE_INCLUDE_DIRS /System/Library/Frameworks)
    FIND_LIBRARY(OpenGL_LIBRARY OpenGL )
    MARK_AS_ADVANCED (OpenGL_LIBRARY)
    SET(OPENGL_LIBS ${OpenGL_LIBRARY})
ELSE (APPLE)
    find_package(OpenGL REQUIRED)
    # list(APPEND SRE_INCLUDE_DIRS ${OPENGL_INCLUDE_DIR})
    IF(UNIX)
        find_package(GLEW REQUIRED)
    ELSE(UNIX)
        find_package(glew REQUIRED)
    ENDIF(UNIX)
    list(APPEND EXTRA_INCLUDE ${GLEW_INCLUDE_DIR})
    SET(OPENGL_LIBS ${OPENGL_LIBRARY} ${GLEW_LIBRARY})
ENDIF (APPLE)

find_package(SDL2_IMAGE REQUIRED)
list(APPEND SRE_INCLUDE_DIRS ${SDL2_IMAGE_INCLUDE_DIRS})

option(USE_OPENVR "Enable OpenVR" OFF)
set(OPENVR_LIB)

# add submodules
add_subdirectory(submodules)

# prepare include directory list
set(SRE_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/include ${EXTRA_INCLUDE} ${SRE_INCLUDE_DIRS} CACHE PATH "" FORCE)
include_directories(${SRE_INCLUDE})

# prepare static library list
set(SRE_LIBRARIES ${OPENGL_LIBS} ${SDL2_LIBRARY} ${SDL2_IMAGE_LIBRARIES} ${OPENVR_LIB})

# prepare dynamic library list

set(SRE_LIBRARIES_SHARED ${SDL2_IMAGE_SHARED_LIBS} ${SDL2_LIBRARY_SHARED})

#######################################################################
# Optional dependencies
#######################################################################
IF (USE_OPENVR)
    find_package(OpenVR REQUIRED)
    list(APPEND SRE_INCLUDE_DIRS ${OPENVR_INCLUDE_DIRS})
	IF(UNIX)
		add_definitions(-DSRE_OPENVR)
	ELSE(UNIX)	
		add_definitions(/DSRE_OPENVR)
	ENDIF(UNIX)
    IF(APPLE)
        if(${OPENVR_PLATFORM} MATCHES "osx32")
            set(OPENVR_LIB "${OPENVR_ROOT_DIR}/lib/${OPENVR_PLATFORM}/libopenvr_api.dylib")
        else()
            set(OPENVR_LIB "${OPENVR_ROOT_DIR}/lib/${OPENVR_PLATFORM}/OpenVR.framework")
        endif()
        MESSAGE("${OPENVR_LIB}")
    ELSE(APPLE)
        set(OPENVR_LIB "${OPENVR_ROOT_DIR}/lib/${OPENVR_PLATFORM}/openvr_api.lib")
    ENDIF(APPLE)

    set(SRE_LIBRARIES SRE    ${SRE_LIBRARIES} ${OPENVR_LIB})
	set(SRE_LIBRARIES_SHARED ${SDL2_IMAGE_SHARED_LIBS} "${OPENVR_ROOT_DIR}/bin/${OPENVR_PLATFORM}/openvr_api.dll")
ENDIF (USE_OPENVR)

#######################################################################
# Create lib
#######################################################################
add_subdirectory(src)

#######################################################################
# Create targets
#######################################################################
SET(USE_SRE_EXAMPLES ON CACHE BOOL "Enable examples for SRE")
if (USE_SRE_EXAMPLES)
    add_subdirectory(examples)
endif(USE_SRE_EXAMPLES)

SET(USE_SRE_TEST_AND_UTILS OFF CACHE BOOL "Enable Tests and Utils for SRE")
IF (USE_SRE_TEST_AND_UTILS)
    add_subdirectory(test)
    add_subdirectory(utils)
ENDIF(USE_SRE_TEST_AND_UTILS)
