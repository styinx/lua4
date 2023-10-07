project(lua)

cmake_minimum_required(VERSION 3.21)

#
# Functions
#

# 'message' wrapper
function(print what)
    message("   ${what}")
endfunction()


#
# Paths
#

# Local paths
set(DIR_ROOT "${PROJECT_SOURCE_DIR}")
set(DIR_INCLUDE "${DIR_ROOT}/include")
set(DIR_SOURCE "${DIR_ROOT}/source")
set(DIR_LIBRARY "${DIR_ROOT}/lib")
set(DIR_TEST "${DIR_ROOT}/test")

# VCPKG paths
set(DIR_VCPKG_INSTALL "${DIR_VCPKG_ROOT}/installed/${VCPKG_TARGET_TRIPLET}")
set(DIR_VCPKG_INCLUDE "${DIR_VCPKG_INSTALL}/include")
set(DIR_VCPKG_LIBRARY "${DIR_VCPKG_INSTALL}/lib")
set(DIR_VCPKG_LIBRARY_DEBUG "${DIR_VCPKG_INSTALL}/debug/lib")

# Binary output paths
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${DIR_LIBRARY}/deb")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG "${DIR_LIBRARY}/deb")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${DIR_ROOT})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${DIR_LIBRARY}/rel")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE "${DIR_LIBRARY}/rel")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${DIR_ROOT})


#
# CXX
#

set(CMAKE_CXX_STANDARD 17)

if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang") # Clang
    #
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU") # GCC
    set(COMPILER_FLAGS "")
    set(COMPILER_FLAGS_DEBUG "-MTd")
    set(COMPILER_FLAGS_RELEASE "-O3 -DNDEBUG -MT")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC") # MSVC
    set(COMPILER_FLAGS "")
    set(COMPILER_FLAGS_DEBUG "/Zi /Ob0 /Od /RTC1 /MTd")
    set(COMPILER_FLAGS_RELEASE "/O2 /Ob2 /DNDEBUG /MT")
endif ()

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${COMPILER_FLAGS} ${COMPILER_FLAGS_DEBUG}")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${COMPILER_FLAGS} ${COMPILER_FLAGS_RELEASE}")


# Dependencies


# CMAKE

set(INCLUDE_DIRECTORIES
        ${DIR_INCLUDE}
        ${DIR_SOURCE}

        ${DIR_VCPKG_INCLUDE})

include_directories(${INCLUDE_DIRECTORIES}
    "src"
    "src/lib"
    "src/luac"
)

if (CMAKE_BUILD_TYPE EQUAL "DEBUG")
    set(LIBRARY_DIRECTORIES
            ${DIR_VCPKG_LIBRARY}
            ${DIR_VCPKG_LIBRARY}/deb)
else()
    set(LIBRARY_DIRECTORIES
            ${DIR_VCPKG_LIBRARY}
            ${DIR_VCPKG_LIBRARY}/rel)
endif()

link_directories(${LIBRARY_DIRECTORIES})


# Sources

set(SOURCES
    src/lapi.c
    src/lcode.c
    src/ldebug.c
    src/ldo.c
    src/lfunc.c
    src/lgc.c
    src/llex.c
    src/lmem.c
    src/lobject.c
    src/lparser.c
    src/lstate.c
    src/lstring.c
	src/ltable.c
	src/ltests.c
	src/ltm.c
	src/lundump.c
	src/lvm.c
	src/lzio.c
)

set(SOURCES_LIB
    "src/lib/lauxlib.c"
    "src/lib/lbaselib.c"
    "src/lib/ldblib.c"
    "src/lib/liolib.c"
    "src/lib/lmathlib.c"
    "src/lib/lstrlib.c"
)

set(SOURCES_LUAC
    "src/luac/print.c"
    "src/luac/dump.c"
    "src/luac/opt.c"
    #"src/luac/stubs.c"
)

source_group("src"      FILES ${SOURCES})
source_group("src/lib"  FILES ${SOURCES_LIB})
source_group("src/luac" FILES ${SOURCES_LUAC})

# Binaries

add_library(lua ${SOURCES})
add_library(lualib ${SOURCES_LIB})

add_executable(luai src/lua/lua.c)
add_executable(luac src/luac/luac.c ${SOURCES_LUAC})

target_link_libraries(luai lualib lua m)
target_link_libraries(luac lualib lua)