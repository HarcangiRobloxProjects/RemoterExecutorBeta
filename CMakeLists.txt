cmake_minimum_required(VERSION 3.10)
project(Remoter)

# Enable MASM immediately after project
enable_language(ASM_MASM)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Force static linking of VC++ runtime (/MT)
if(MSVC)
    set(CompilerFlags
        CMAKE_CXX_FLAGS
        CMAKE_CXX_FLAGS_DEBUG
        CMAKE_CXX_FLAGS_RELEASE
        CMAKE_C_FLAGS
        CMAKE_C_FLAGS_DEBUG
        CMAKE_C_FLAGS_RELEASE
    )
    foreach(CompilerFlag ${CompilerFlags})
        string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
    endforeach()
endif()
# Dependencies
include(FetchContent)

# ---- xxHash (for RSB1 hashing) ----
FetchContent_Declare(
    xxhash
    GIT_REPOSITORY https://github.com/Cyan4973/xxHash.git
    GIT_TAG v0.8.3
    SOURCE_SUBDIR cmake/cmake
)
set(XXHASH_BUILD_XXHSUM OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(xxhash)

# ---- ZSTD (for RSB1 bytecode encoding) ----
FetchContent_Declare(
    zstd
    GIT_REPOSITORY https://github.com/facebook/zstd.git
    GIT_TAG v1.5.6
    SOURCE_SUBDIR build/cmake
)
set(ZSTD_BUILD_PROGRAMS OFF CACHE BOOL "" FORCE)
set(ZSTD_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(ZSTD_BUILD_SHARED OFF CACHE BOOL "" FORCE)
set(ZSTD_BUILD_STATIC ON CACHE BOOL "" FORCE)
set(ZSTD_MULTITHREAD_SUPPORT OFF CACHE BOOL "" FORCE)
set(ZSTD_BUILD_CONTRIB OFF CACHE BOOL "" FORCE)
set(ZSTD_LEGACY_SUPPORT OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(zstd)

# ---- BLAKE3 (for bytecode signing) ----
FetchContent_Declare(
    blake3
    GIT_REPOSITORY https://github.com/BLAKE3-team/BLAKE3.git
    GIT_TAG 1.8.2
    SOURCE_SUBDIR c
)
set(BLAKE3_FETCH_CONTENT ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(blake3)

# ---- cpp-httplib ----
FetchContent_Declare(
    httplib
    GIT_REPOSITORY https://github.com/yhirose/cpp-httplib.git
    GIT_TAG v0.18.3
)
set(HTTPLIB_COMPILE OFF CACHE BOOL "" FORCE)
set(HTTPLIB_USE_OPENSSL_IF_AVAILABLE OFF CACHE BOOL "" FORCE)
set(HTTPLIB_REQUIRE_OPENSSL OFF CACHE BOOL "" FORCE)
set(HTTPLIB_USE_BROTLI_IF_AVAILABLE OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(httplib)

# ---- nlohmann/json ----
FetchContent_Declare(
    json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.3
)
set(JSON_BuildTests OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(json)

# ---- Luau Compiler ----
FetchContent_Declare(
    luau
    GIT_REPOSITORY https://github.com/luau-lang/luau.git
    GIT_TAG 0.650
)
set(LUAU_BUILD_CLI OFF CACHE BOOL "" FORCE)
set(LUAU_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(LUAU_BUILD_WEB OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(luau)

# Identify core source files (FLAT STRUCTURE)
# Explicitly using main file to avoid entry point conflicts
set(CORE_SOURCES 
    "src/exports.cpp" 
    "src/syscalls.asm"
)

# Force MASM for the syscall file
set_source_files_properties("src/syscalls.asm" PROPERTIES LANGUAGE ASM_MASM)

# Include directories
include_directories(src)

# Build the Core DLL
add_library(Remoter SHARED ${CORE_SOURCES})
target_compile_definitions(Remoter PRIVATE RBLX_EXPORT WIN32_LEAN_AND_MEAN NOMINMAX)
set_target_properties(Remoter PROPERTIES LINKER_LANGUAGE CXX)

# Link Luau Compiler + Ast libraries
target_link_libraries(Remoter PRIVATE 
    kernel32 user32 winhttp ws2_32
    Luau.Compiler Luau.Ast
    libzstd_static
    blake3
    xxhash::xxhash
    httplib::httplib
    nlohmann_json::nlohmann_json
)

# Add Luau include paths
target_include_directories(Remoter PRIVATE
    ${luau_SOURCE_DIR}/Compiler/include
    ${luau_SOURCE_DIR}/Ast/include
    ${luau_SOURCE_DIR}/Common/include
    ${zstd_SOURCE_DIR}/lib
    ${zstd_SOURCE_DIR}/lib/common
    ${blake3_SOURCE_DIR}/c
    ${xxhash_SOURCE_DIR}
)
