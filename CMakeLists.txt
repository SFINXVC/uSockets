cmake_minimum_required(VERSION 3.15)

project(uSockets)

option(LIBUS_WITH_LTO "Enable Link Time Optimization" OFF)
option(LIBUS_WITH_BORINGSSL "Enable BoringSSL support" ON)
option(LIBUS_WITH_OPENSSL "Enable OpenSSL support" OFF)
option(LIBUS_WITH_WOLFSSL "Enable WolfSSL support" OFF)
option(LIBUS_WITH_IO_URING "Enable io_uring support" OFF)
option(LIBUS_WITH_LIBUV "Enable libuv support" OFF)
option(LIBUS_WITH_ASIO "Enable Asio support" OFF)
option(LIBUS_WITH_GCD "Enable GCD support" OFF)
option(LIBUS_WITH_ASAN "Enable AddressSanitizer" OFF)
option(LIBUS_WITH_QUIC "Enable QUIC support" OFF)

option(LIBUS_BUILD_EXAMPLES "Build uSockets example projects" ON)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_compile_options(-O3)

file(GLOB SOURCES
    src/*.c
    src/eventing/*.c
    src/crypto/*.c
    src/io_uring/*.c
)

add_subdirectory(libuv)

set(LIBUV_BUILD_SHARED OFF)
add_library(uSockets STATIC ${SOURCES})

target_link_libraries(uSockets PUBLIC uv)
target_include_directories(uSockets PUBLIC src/ libuv/include)

if(LIBUS_WITH_LTO)
    include(CheckIPOSupported)
    check_ipo_supported(RESULT LTO_SUPPORTED OUTPUT error)

    if (LTO_SUPPORTED)
        message("LTO is supported, enabling -flto")
        set_property(TARGET uSockets PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
    endif ()
elseif(LIBUS_WITH_BORINGSSL)
    find_package(Threads REQUIRED)
    add_subdirectory(BoringSSL)

    target_compile_definitions(uSockets PRIVATE LIBUS_USE_OPENSSL)
    target_link_libraries(uSockets PUBLIC BoringSSL::ssl BoringSSL::crypto)
elseif(LIBUS_WITH_OPENSSL)
    find_package(OpenSSL REQUIRED)

    target_compile_definitions(uSockets PRIVATE LIBUS_USE_OPENSSL)
    target_link_libraries(uSockets PUBLIC openssl::openssl)
else()
    target_compile_definitions(uSockets PRIVATE LIBUS_NO_SSL)
endif()

if(LIBUS_WITH_WOLFSSL)
    find_package(wolfssl REQUIRED)

    target_compile_definitions(uSockets PRIVATE LIBUS_USE_WOLFSSL)
    target_link_libraries(uSockets PUBLIC wolfssl::wolfssl)
endif()

if(LIBUS_WITH_IO_URING)
    target_compile_definitions(uSockets PRIVATE LIBUS_USE_IO_URING)
endif()

if(LIBUS_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()