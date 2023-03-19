# Copyright(c) 2019 spdlog authors Distributed under the MIT License (http://opensource.org/licenses/MIT)

cmake_minimum_required(VERSION 3.10)
project(spdlog_examples CXX)

if(NOT TARGET spdlog)
    # Stand-alone build
    find_package(spdlog REQUIRED)
endif()

# ---------------------------------------------------------------------------------------
# Example of using pre-compiled library
# ---------------------------------------------------------------------------------------
add_executable(example example.cpp)
target_link_libraries(example PRIVATE spdlog::spdlog $<$<BOOL:${MINGW}>:ws2_32>)

## FEC Optimizations begin ---------------------------------
option(WB_ENABLE_SIMD_OPTIMIZATIONS "Enable NEON on ARM and AVX2 on X86 if the platform/compiler supports it" ON)
set(WIFIBROADCAST_FEC_OPTIMIZATION_FLAGS_X86 -mavx2 -faligned-new=256)
include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-mavx2" COMPILER_SUPPORTS_MAVX2)
# AVX2 if supported and option WB_ENABLE_SIMD_OPTIMIZATIONS is true
if(COMPILER_SUPPORTS_MAVX2)
    message(STATUS "Compiler supports AVX2")
    if(WB_ENABLE_SIMD_OPTIMIZATIONS)
        message(STATUS "WB compile with AVX2")
        target_compile_options(example PUBLIC ${WIFIBROADCAST_FEC_OPTIMIZATION_FLAGS_X86})
        # I do not know why, but using target_compile_options seems to be not enough ...
        add_compile_options(${WIFIBROADCAST_FEC_OPTIMIZATION_FLAGS_X86})
        target_compile_definitions(example PUBLIC WIFIBROADCAST_HAS_X86_AVX2)
    endif()
endif()
## FEC Optimizations end ---------------------------------

# ---------------------------------------------------------------------------------------
# Example of using header-only library
# ---------------------------------------------------------------------------------------
if(SPDLOG_BUILD_EXAMPLE_HO)
    add_executable(example_header_only example.cpp)
    target_link_libraries(example_header_only PRIVATE spdlog::spdlog_header_only)
endif()
