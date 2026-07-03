cmake_minimum_required(VERSION 3.20)
project(GhostLight LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 1. Setup Paths
set(EXTERN_DIR "${CMAKE_CURRENT_SOURCE_DIR}/extern")
set(ULTRALIGHT_DIR "${EXTERN_DIR}/ultralight")
set(ARCHIVE_PATH "${ULTRALIGHT_DIR}/sdk.7z")

# 2. Determine OS-specific URL
if(APPLE)
    set(SDK_URL "https://ultralight-sdk.sfo2.cdn.digitaloceanspaces.com/ultralight-sdk-latest-mac-x64.7z")
elseif(WIN32)
    set(SDK_URL "https://ultralight-sdk.sfo2.cdn.digitaloceanspaces.com/ultralight-sdk-latest-win-x64.7z")
else()
    set(SDK_URL "https://ultralight-sdk.sfo2.cdn.digitaloceanspaces.com/ultralight-sdk-latest-linux-x64.7z")
endif()

# 3. Download SDK
file(DOWNLOAD ${SDK_URL} ${ARCHIVE_PATH} SHOW_PROGRESS)

# 4. Extract using py7zr CLI directly
# This assumes 'py7zr' is in your system PATH after 'pip install py7zr'
execute_process(
    COMMAND py7zr x ${ARCHIVE_PATH}
    WORKING_DIRECTORY ${ULTRALIGHT_DIR}
    RESULT_VARIABLE EXTRACT_RESULT
)

if(NOT EXTRACT_RESULT EQUAL 0)
    message(FATAL_ERROR "Extraction failed! Ensure py7zr is installed and in your PATH.")
endif()

# 5. Include and Link setup
include_directories(${ULTRALIGHT_DIR}/include)
link_directories(${ULTRALIGHT_DIR}/bin)

# 6. Core Executable Definition
add_executable(ghostlight_core src/main.cpp)

# 7. Linking
target_link_libraries(ghostlight_core PRIVATE 
    Ultralight
    UltralightCore
    AppCore
)

# 8. Runtime dependency copy
add_custom_command(TARGET ghostlight_core POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    "${ULTRALIGHT_DIR}/bin/"
    $<TARGET_FILE_DIR:ghostlight_core>
)
