# CMakeLists.txt for libargon2

cmake_minimum_required(VERSION 3.10)
project(argon2 LANGUAGES C)

set(ARGON2_SOURCES
    src/argon2.c
    src/core.c
    src/blake2/blake2b.c
    src/thread.c
    src/encoding.c
)

# Use optimized implementation on x86/x64, reference on other platforms
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64|AMD64|i[3-6]86)")
    list(APPEND ARGON2_SOURCES src/opt.c)
else()
    list(APPEND ARGON2_SOURCES src/ref.c)
endif()

add_library(argon2 STATIC ${ARGON2_SOURCES})
add_library(argon2::argon2 ALIAS argon2)

target_include_directories(argon2
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Threading support
find_package(Threads)
if(Threads_FOUND)
    target_link_libraries(argon2 PRIVATE Threads::Threads)
else()
    target_compile_definitions(argon2 PRIVATE ARGON2_NO_THREADS)
endif()

# Compiler settings
target_compile_options(argon2 PRIVATE
    $<$<C_COMPILER_ID:GNU,Clang,AppleClang>:-O3 -Wall>
)

if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64|AMD64|i[3-6]86)")
    target_compile_options(argon2 PRIVATE
        $<$<C_COMPILER_ID:GNU,Clang,AppleClang>:-march=native>
    )
endif()
