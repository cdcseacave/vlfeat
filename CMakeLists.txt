# CMake powered build system for vlfeat
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
# as ${vlfeat_SOURCE_DIR} and to the root binary directory as
# ${vlfeat_BINARY_DIR}.
PROJECT(vlfeat)

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
ConfigLibrary()

# List config options
SET(vlfeat_USE_OPENMP ON CACHE BOOL "Enable OpenMP library")
SET(BUILD_APPS ON CACHE BOOL "Build applications")

INCLUDE_DIRECTORIES("${vlfeat_SOURCE_DIR}")

# Find required packages
if(vlfeat_USE_OPENMP)
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

if (BUILD_SHARED_LIBS)
	ADD_DEFINITIONS(-DVL_BUILD_DLL)
endif()

# Set defines
SET(vlfeat_DEFINITIONS "")
if(MSVC)
	LIST(APPEND vlfeat_DEFINITIONS "/wd4231") # disable warnings on extern before template instantiation
	LIST(APPEND vlfeat_DEFINITIONS "/wd4251") # disable warnings on needs to have dll-interface to be used by clients of class ...
	LIST(APPEND vlfeat_DEFINITIONS "/wd4308") # for negative integral constant converted to unsigned type
	LIST(APPEND vlfeat_DEFINITIONS "/wd4396") # for ignored inlines
	LIST(APPEND vlfeat_DEFINITIONS "/wd4503") # for decorated name length exceeded, name was truncated
	LIST(APPEND vlfeat_DEFINITIONS "/wd4661") # disable warnings on no suitable definition provided for explicit template instantiation request
	LIST(APPEND vlfeat_DEFINITIONS "/wd4996") # for deprecated functions
endif()
if(ENABLE_SSE2)
	LIST(APPEND vlfeat_DEFINITIONS -D__SSE2__)
else()
	LIST(APPEND vlfeat_DEFINITIONS -DVL_DISABLE_SSE2)
endif()
if(ENABLE_AVX)
	LIST(APPEND vlfeat_DEFINITIONS -D__AVX__)
else()
	LIST(APPEND vlfeat_DEFINITIONS -DVL_DISABLE_AVX)
endif()
ADD_DEFINITIONS(${vlfeat_DEFINITIONS})

# Add SfM libraries
ADD_SUBDIRECTORY(vl)

# Add applications
if (BUILD_APPS)
	ADD_SUBDIRECTORY(src)
endif()

# Add all targets to the build-tree export set
export(TARGETS vlfeat FILE "${PROJECT_BINARY_DIR}/vlfeatTargets.cmake")

# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry)
export(PACKAGE vlfeat)
 
# Create the vlfeatConfig.cmake and vlfeatConfigVersion files
# ... for the build tree
set(CONF_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}" "${PROJECT_BINARY_DIR}")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/CMake/vlfeatConfig.cmake.in" "${PROJECT_BINARY_DIR}/vlfeatConfig.cmake" @ONLY)
# ... for the install tree
set(CONF_INCLUDE_DIRS "${INSTALL_INCLUDEROOT_DIR}" "${INSTALL_INCLUDE_DIR}")
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/CMake/vlfeatConfig.cmake.in" "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/vlfeatConfig.cmake" @ONLY)
# ... for both
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/CMake/vlfeatConfigVersion.cmake.in" "${PROJECT_BINARY_DIR}/vlfeatConfigVersion.cmake" @ONLY)
 
# Install the vlfeatConfig.cmake and vlfeatConfigVersion.cmake
install(FILES
	"${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/vlfeatConfig.cmake"
	"${PROJECT_BINARY_DIR}/vlfeatConfigVersion.cmake"
	DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

# Install the export set for use with the install-tree
install(EXPORT vlfeatTargets DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

# uninstall target
configure_file(
	"${CMAKE_CURRENT_SOURCE_DIR}/CMake/cmake_uninstall.cmake.in"
	"${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
	IMMEDIATE @ONLY)

add_custom_target(uninstall
	COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
