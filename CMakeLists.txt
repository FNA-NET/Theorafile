# CMake Project for theorafile
# Written by @NeroBurner
cmake_minimum_required(VERSION 3.10)
project(theorafile C)

# Options
option(BUILD_SHARED_LIBS "Build shared library" ON)

# C99 Requirement
if(${CMAKE_VERSION} VERSION_LESS "3.1.3")
	message(WARNING "Your CMake version is too old, set -std=c99 yourself!")
else()
	set(CMAKE_C_STANDARD 99)
	set(CMAKE_C_EXTENSIONS OFF)
endif()

# Version
SET(LIB_MAJOR_VERSION "0")
SET(LIB_MINOR_VERSION "25")
SET(LIB_REVISION "06")
SET(LIB_VERSION "${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION}.${LIB_REVISION}")

# Build Type
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	# By default, we use Release
	message(STATUS "Setting build type to 'Release' as none was specified.")
	set(CMAKE_BUILD_TYPE "Release" CACHE
		STRING "Choose the type of build." FORCE
	)

	# Set the possible values of build type for cmake-gui
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY
		STRINGS "Debug" "Release" "RelWithDebInfo"
	)
endif()

# Add our repository's module path to CMake's module path list
set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

# Platform Flags
add_definitions(-DOC_ARM_ASM -DOC_ARM_ASM_EDSP -DOC_ARM_ASM_MEDIA -DOC_ARM_ASM_NEON)

# Source lists
set(SRC_FILES
	theorafile.c
	lib/ogg/bitwise.c
	lib/ogg/framing.c
	lib/vorbis/analysis.c
	lib/vorbis/bitrate.c
	lib/vorbis/block.c
	lib/vorbis/codebook.c
	lib/vorbis/envelope.c
	lib/vorbis/floor0.c
	lib/vorbis/floor1.c
	lib/vorbis/vinfo.c
	lib/vorbis/lookup.c
	lib/vorbis/lpc.c
	lib/vorbis/lsp.c
	lib/vorbis/mapping0.c
	lib/vorbis/mdct.c
	lib/vorbis/psy.c
	lib/vorbis/registry.c
	lib/vorbis/res0.c
	lib/vorbis/sharedbook.c
	lib/vorbis/smallft.c
	lib/vorbis/synthesis.c
	lib/vorbis/window.c
	lib/theora/apiwrapper.c
	lib/theora/bitpack.c
	lib/theora/decapiwrapper.c
	lib/theora/decinfo.c
	lib/theora/decode.c
	lib/theora/dequant.c
	lib/theora/fragment.c
	lib/theora/huffdec.c
	lib/theora/idct.c
	lib/theora/tinfo.c
	lib/theora/internal.c
	lib/theora/quant.c
	lib/theora/state.c
	lib/theora/arm-intrinsics/armcpu.c
	lib/theora/arm-intrinsics/armfrag.c
	lib/theora/arm-intrinsics/armidct.c
	lib/theora/arm-intrinsics/armloop.c
	lib/theora/arm-intrinsics/armstate.c
)

add_library(theorafile ${SRC_FILES})

target_link_libraries(theorafile PRIVATE -fpic -fPIC -O3 -lm)

# Only disable DebugConfiguration in release builds
if(NOT FORCE_ENABLE_DEBUGCONFIGURATION)
	target_compile_definitions(theorafile PRIVATE $<$<CONFIG:Release>:THEORAFILEDISABLE_DEBUGCONFIGURATION>)
endif()

# Theorafile folders as includes, for other targets to consume
target_include_directories(theorafile PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/ogg>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/theora>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/lib/vorbis>
)

# Soname
set_target_properties(theorafile PROPERTIES OUTPUT_NAME "theorafile"
	VERSION ${LIB_VERSION}
	SOVERSION ${LIB_MAJOR_VERSION}
)
