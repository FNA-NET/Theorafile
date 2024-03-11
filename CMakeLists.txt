# CMake Project for theorafile
# Written by @NeroBurner
cmake_minimum_required(VERSION 2.8.12)
project(theorafile C)

# Options
option(BUILD_SHARED_LIBS "Build shared library" ON)
if(WIN32)
endif()

# C99 Requirement
if(${CMAKE_VERSION} VERSION_LESS "3.1.3")
	message(WARNING "Your CMake version is too old, set -std=c99 yourself!")
else()
	set(CMAKE_C_STANDARD 99)
	set(CMAKE_C_EXTENSIONS OFF)
endif()

# Version
SET(LIB_MAJOR_VERSION "0")
SET(LIB_MINOR_VERSION "21")
SET(LIB_REVISION "11")
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

set(CMAKE_SKIP_BUILD_RPATH FALSE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

# Platform Flags
if(APPLE)
	set(CMAKE_MACOSX_RPATH ON)
	set(CMAKE_INSTALL_RPATH "@loader_path")
	set(CMAKE_OSX_DEPLOYMENT_TARGET 11.0)
elseif(UNIX)
	set(CMAKE_INSTALL_RPATH $ORIGIN)
elseif(WIN32)
	# "libtheorafile.dll"
	set(CMAKE_SHARED_LIBRARY_PREFIX "lib")

	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		# 64 bits
		add_definitions(-DOC_X86_64_ASM)
	elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
		# 32 bits
		add_definitions(-DOC_X86_ASM)
	endif()

endif()

# Helper function for finding/installing MinGW libs
if(INSTALL_MINGW_DEPENDENCIES)
	include(cmake/install_shared_libs.cmake)
endif()

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
)

if (MSVC)
	list(APPEND SRC_FILES
		lib/theora/x86_vc/mmxfrag.c
		lib/theora/x86_vc/mmxidct.c
		lib/theora/x86_vc/mmxstate.c
		lib/theora/x86_vc/x86cpu.c
		lib/theora/x86_vc/x86state.c
	)
else()
	list(APPEND SRC_FILES
		lib/theora/x86/mmxfrag.c
		lib/theora/x86/mmxidct.c
		lib/theora/x86/mmxstate.c
		lib/theora/x86/x86state.c
	)
endif()

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

# MinGW builds should statically link libgcc
if(MINGW)
	target_link_libraries(theorafile PRIVATE -static-libgcc)
endif()

# Soname
set_target_properties(theorafile PROPERTIES OUTPUT_NAME "theorafile"
	VERSION ${LIB_VERSION}
	SOVERSION ${LIB_MAJOR_VERSION}
)

# define install directories
# on mingw-w64 cross compilation $CMAKE_INSTALL_LIBDIR is set to an absolute
# path. Work around that by hard coding the directories on windows
if(WIN32)
	set(Theorafile_INSTALL_INCLUDEDIR include)
	set(Theorafile_INSTALL_BINDIR bin)
	set(Theorafile_INSTALL_LIBDIR lib)
else()
	include(GNUInstallDirs)
	set(Theorafile_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR})
	set(Theorafile_INSTALL_BINDIR ${CMAKE_INSTALL_BINDIR})
	set(Theorafile_INSTALL_LIBDIR ${CMAKE_INSTALL_LIBDIR})
endif()

# Installation

# Public Headers...
install(
	DIRECTORY include/
	DESTINATION ${Theorafile_INSTALL_INCLUDEDIR}
)
# Libraries...
install(
	TARGETS ${PROJECT_NAME}
	EXPORT ${PROJECT_NAME}-targets
	INCLUDES DESTINATION ${Theorafile_INSTALL_INCLUDEDIR}
	RUNTIME DESTINATION ${Theorafile_INSTALL_BINDIR}
	LIBRARY DESTINATION ${Theorafile_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${Theorafile_INSTALL_LIBDIR}
)

# Generate a pkgconfig file
include(cmake/JoinPaths.cmake)
join_paths(THEORAFILEPKGCONF_LIBDIR "\${prefix}" "${CMAKE_INSTALL_LIBDIR}")
join_paths(THEORAFILEPKGCONF_INCLUDEDIR "\${prefix}" "${CMAKE_INSTALL_INCLUDEDIR}")
configure_file(
	"${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}.pc.in"
	${PROJECT_BINARY_DIR}/generated/${PROJECT_NAME}.pc
	@ONLY
)
install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}.pc
	DESTINATION ${Theorafile_INSTALL_LIBDIR}/pkgconfig
)

# Generate cmake-config file, install CMake files
include(CMakePackageConfigHelpers)
configure_package_config_file(
	cmake/config.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake
	INSTALL_DESTINATION ${Theorafile_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake
	DESTINATION ${Theorafile_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
install(
	EXPORT ${PROJECT_NAME}-targets
	NAMESPACE ${PROJECT_NAME}::
	DESTINATION ${Theorafile_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
