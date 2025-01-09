cmake_minimum_required(VERSION 3.15)
project(hsdis)

# options for users
set(HSDIS_JDK_REF a87bc7e4f0e797a108f447a1c9801abe39b700da
    CACHE STRING "git ref to download hsdis sources from")
set(HSDIS_CAPSTONE_REF b102f1b89e0455c072a751d287ab64378c14205f
    CACHE STRING "git ref to fetch capstone from")
set(HSDIS_ARCH X64
    CACHE STRING "hsdis target architecture")

# internal settings
set(CMAKE_POSITION_INDEPENDENT_CODE ON) # needed for linux
 # turn off architecture support by default, to get a smaller capstone library
set(CAPSTONE_ARCHITECTURE_DEFAULT OFF)

# set architecture specific options. Only X64 or AArch64 for now
if(${HSDIS_ARCH} STREQUAL X64)
    set(CAPSTONE_X86_SUPPORT ON)
    set(HSDIS_CAPSTONE_ARCH CS_ARCH_X86)
    set(HSDIS_CAPSTONE_MODE CS_MODE_64)
    set(HSDIS_LIB_SUFFIX amd64)
elseif(${HSDIS_ARCH} STREQUAL AArch64)
    set(CAPSTONE_AARCH64_SUPPORT ON)
    set(HSDIS_CAPSTONE_ARCH CS_ARCH_AARCH64)
    set(HSDIS_CAPSTONE_MODE CS_MODE_LITTLE_ENDIAN)
    set(HSDIS_LIB_SUFFIX aarch64)
else()
    message(FATAL_ERROR "Unknown architecture: ${HSDIS_ARCH}")
endif()

# fetch and build capstone
include(FetchContent)
message(STATUS "Fetching capstone (ref=${HSDIS_CAPSTONE_REF})...")
FetchContent_Declare(
    capstone
    GIT_REPOSITORY https://github.com/capstone-engine/capstone
    GIT_TAG ${HSDIS_CAPSTONE_REF})
FetchContent_MakeAvailable(capstone)

# build hsdis
# 1. find jni.h
find_package(JNI REQUIRED)

if (JNI_FOUND)
    message (STATUS "JNI_INCLUDE_DIRS=${JNI_INCLUDE_DIRS}")
    message (STATUS "JNI_LIBRARIES=${JNI_LIBRARIES}")
endif()

# 2. download source files
set(HSDIS_SOURCE_ROOT_URL
    https://raw.githubusercontent.com/openjdk/jdk/${HSDIS_JDK_REF}/src/utils/hsdis)
file(DOWNLOAD
    ${HSDIS_SOURCE_ROOT_URL}/capstone/hsdis-capstone.c
    ${CMAKE_SOURCE_DIR}/src/hsdis-capstone.c)
file(DOWNLOAD
    ${HSDIS_SOURCE_ROOT_URL}/hsdis.h
    ${CMAKE_SOURCE_DIR}/src/hsdis.h)

# 3. fixup capstone.h include
file(READ src/hsdis-capstone.c FILE_CONTENTS)
string(REPLACE "#include <capstone.h>" "#include <capstone/capstone.h>"
       FILE_CONTENTS "${FILE_CONTENTS}")
file(WRITE src/hsdis-capstone.c "${FILE_CONTENTS}")

# 4. add hsdis shared library target
add_library(hsdis SHARED src/hsdis-capstone.c)

# 5. configure target
target_link_libraries(hsdis PRIVATE JNI::JNI capstone)
target_include_directories(hsdis PUBLIC src)
target_compile_definitions(hsdis
  PRIVATE 
    CAPSTONE_ARCH=${HSDIS_CAPSTONE_ARCH}
    CAPSTONE_MODE=${HSDIS_CAPSTONE_MODE})

message(STATUS "Output library name: hsdis-${HSDIS_LIB_SUFFIX}")
set_target_properties(hsdis
  PROPERTIES
    OUTPUT_NAME hsdis-${HSDIS_LIB_SUFFIX}
    PREFIX "")

# 6. generate install target
install(TARGETS hsdis)
