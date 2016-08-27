# CMake powered build system for VLFeat
# First draft: 22 august 2012

########################################################################
#
# Project-wide settings
CMAKE_MINIMUM_REQUIRED(VERSION 2.6.2)

# Find dependencies:
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/CMake/Modules)

# Name of the project.
#
# CMake files in this project can refer to the root source directory
# as ${VLFeat_SOURCE_DIR} and to the root binary directory as
# ${VLFeat_BINARY_DIR}.
PROJECT(VLFeat)

# fix CMake IntDir variable
if(MSVC AND "${MSVC_VERSION}" STRGREATER "1500")
    #SET(COTIRE_INTDIR "$(Platform)/cotire")
    SET(CMAKE_CFG_INTDIR "$(Platform)/$(Configuration)")
endif()

# Define helper functions and macros.
INCLUDE(CMake/Utils.cmake)
INCLUDE(CMake/Cotire.cmake)

# Init sesion with macros defined in Utils.cmake
GetOperatingSystemArchitectureBitness(SYSTEM)
ComposePackageLibSuffix()
ConfigCompilerAndLinker()

# List config options
SET(VLFeat_USE_OPENMP ON CACHE BOOL "Enable OpenMP library")

INCLUDE_DIRECTORIES("${VLFeat_SOURCE_DIR}")

# Find required packages
if(VLFeat_USE_OPENMP)
	FIND_PACKAGE(OpenMP)
	if(OPENMP_FOUND)
		SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
		ADD_DEFINITIONS(-D_USE_OPENMP)
	else()
		ADD_DEFINITIONS(-DVL_DISABLE_THREADS -DVL_DISABLE_OPENMP)
		MESSAGE("-- Can't find OpenMP. Continuing without it.")
	endif()
else()
	ADD_DEFINITIONS(-DVL_DISABLE_THREADS -DVL_DISABLE_OPENMP)
endif()

# Set defines
SET(VLFeat_DEFINITIONS "")
if(MSVC)
	LIST(APPEND VLFeat_DEFINITIONS "/wd4231") # disable warnings on extern before template instantiation
	LIST(APPEND VLFeat_DEFINITIONS "/wd4251") # disable warnings on needs to have dll-interface to be used by clients of class ...
	LIST(APPEND VLFeat_DEFINITIONS "/wd4308") # for negative integral constant converted to unsigned type
	LIST(APPEND VLFeat_DEFINITIONS "/wd4396") # for ignored inlines
	LIST(APPEND VLFeat_DEFINITIONS "/wd4503") # for decorated name length exceeded, name was truncated
	LIST(APPEND VLFeat_DEFINITIONS "/wd4661") # disable warnings on no suitable definition provided for explicit template instantiation request
	LIST(APPEND VLFeat_DEFINITIONS "/wd4996") # for deprecated functions
endif()
if(ENABLE_SSE2)
	LIST(APPEND VLFeat_DEFINITIONS -D__SSE2__)
else()
	LIST(APPEND VLFeat_DEFINITIONS -DVL_DISABLE_SSE2)
endif()
if(ENABLE_AVX)
	LIST(APPEND VLFeat_DEFINITIONS -D__AVX__)
else()
	LIST(APPEND VLFeat_DEFINITIONS -DVL_DISABLE_AVX)
endif()
ADD_DEFINITIONS(${VLFeat_DEFINITIONS})

# Add SfM libraries
ADD_SUBDIRECTORY(vl)

# Add applications
ADD_SUBDIRECTORY(src)
