cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW)
cmake_policy(SET CMP0042 NEW)
if(POLICY CMP0135)
  cmake_policy(SET CMP0135 NEW) # DOWNLOAD_EXTRACT_TIMESTAMP
endif()

project (smartcard_napi)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake/modules/")

# define NPI_VERSION
add_definitions(-DNAPI_VERSION=8 -DNAPI_CPP_EXCEPTIONS)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include_directories(${CMAKE_JS_INC})
include(FindPCSC)

set(SOURCE_FILES
  "src/addon.cpp"
  "src/async_workers.cpp"
  "src/async_workers.h"
  "src/pcsc_card.cpp"
  "src/pcsc_card.h"
  "src/pcsc_context.cpp"
  "src/pcsc_context.h"
  "src/pcsc_errors.h"
  "src/pcsc_reader.cpp"
  "src/pcsc_reader.h"
  "src/reader_monitor.cpp"
  "src/reader_monitor.h"
)

if(UNIX AND NOT APPLE)
  # build shared on linux, to avoid conflicts with libjpeg built into electron
  set(CMAKE_CXX_EXCEPTIONS ON)
  set(CMAKE_CXX_RTTI ON)

  set(ENABLE_SHARED 1)
  set(ENABLE_STATIC 0)
else()
  # build static elsewhere
  set(ENABLE_SHARED 0)
  set(ENABLE_STATIC 1)
endif()


if (MSVC)
  set(CMAKE_CXX_EXCEPTIONS ON)
  set(CMAKE_CXX_RTTI ON)
endif()

if (APPLE)
  set(CMAKE_XCODE_ATTRIBUTE_GCC_ENABLE_CPP_EXCEPTIONS ON)
  set(CMAKE_XCODE_ATTRIBUTE_GCC_ENABLE_CPP_RTTI ON)
endif()


set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "$ORIGIN")

IF(PCSC_INCLUDE_DIR)
  INCLUDE_DIRECTORIES(${PCSC_INCLUDE_DIR})
  LINK_DIRECTORIES(${PCSC_LIBRARY_DIR})
ENDIF(PCSC_INCLUDE_DIR)

add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES} ${CMAKE_JS_SRC})
set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "" SUFFIX ".node")
target_link_libraries(${PROJECT_NAME} ${CMAKE_JS_LIB} ${PCSC_LIBRARIES})
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)

if(UNIX AND NOT APPLE)
  target_compile_options(${PROJECT_NAME} PRIVATE "-fexceptions")
  target_link_options(${PROJECT_NAME} PRIVATE "-fexceptions")
endif()

if(MSVC AND CMAKE_JS_NODELIB_DEF AND CMAKE_JS_NODELIB_TARGET)
  # Generate node.lib
  execute_process(COMMAND ${CMAKE_AR} /def:${CMAKE_JS_NODELIB_DEF} /out:${CMAKE_JS_NODELIB_TARGET} ${CMAKE_STATIC_LINKER_FLAGS})
endif()
