cmake_minimum_required(VERSION 3.1)

project(base64 VERSION 0.0.1)



##
## CONFIGURATION
##
set(BASE64_TARGET_NAME               ${PROJECT_NAME})
set(BASE64_INCLUDE_BUILD_DIR "${PROJECT_SOURCE_DIR}/include")

option(HAVE_AVX2 "Build AVX2" OFF)
option(HAVE_NEON32 "Build neon32" OFF)
option(HAVE_NEON64 "Build neon64" OFF)


##
## TARGET
##
message("Target name: ${BASE64_TARGET_NAME}")
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_library(${BASE64_TARGET_NAME} STATIC
  lib/arch/avx2/codec.c
  lib/arch/generic/codec.c
  lib/arch/neon32/codec.c
  lib/arch/neon64/codec.c
  lib/arch/ssse3/codec.c
  lib/arch/sse41/codec.c
  lib/arch/sse42/codec.c
  lib/arch/avx/codec.c
  lib/lib.c
  lib/codec_choose.c
  lib/tables/tables.c
)
configure_file("${PROJECT_SOURCE_DIR}/config.cmake.in" "${PROJECT_SOURCE_DIR}/lib/config.h")
add_library(${PROJECT_NAME}::${BASE64_TARGET_NAME} ALIAS ${BASE64_TARGET_NAME})
target_include_directories(
    ${BASE64_TARGET_NAME}
    PUBLIC $<BUILD_INTERFACE:${${PROJECT_NAME}_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

set(C_FLAGS "")
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  set(C_FLAGS "-msse3;-mssse3;-mavx")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set(C_FLAGS "-msse3;-mssse3;-mavx")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
  set(C_FLAGS "-msse3;-mssse3;-mavx")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
  set(C_FLAGS "/arch:AVX")
endif()

target_compile_options(${BASE64_TARGET_NAME} PRIVATE ${C_FLAGS})

# INSTALL
install(TARGETS ${PROJECT_NAME}
      EXPORT ${PROJECT_NAME}_Targets
       ARCHIVE DESTINATION lib
       LIBRARY DESTINATION lib
       RUNTIME DESTINATION lib
       )

include(CMakePackageConfigHelpers)

write_basic_package_version_file("${PROJECT_NAME}ConfigVersion.cmake"
                                 VERSION ${PROJECT_VERSION}
                                 COMPATIBILITY SameMajorVersion)

install(DIRECTORY include DESTINATION .)
