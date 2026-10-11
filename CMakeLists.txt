cmake_minimum_required(VERSION 3.20)
project(Vst3SinePlugin VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Platform detection
if(CMAKE_HOST_APPLE)
    set(PLATFORM_NAME Mac)
elseif(CMAKE_HOST_UNIX)
    set(PLATFORM_NAME Linux)
else()
    set(PLATFORM_NAME Windows)
endif()

# Find JUCE
# Option 1: System installation (via Homebrew on macOS)
find_package(JUCE CONFIG QUIET)

# Option 2: Download JUCE
if(NOT JUCE_FOUND)
    message(STATUS "JUCE not found, downloading from GitHub...")
    
    # Create build directory for JUCE
    file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/juce)
    
    # Download JUCE source
    file(DOWNLOAD
        https://github.com/juce/juce/archive/refs/tags/7.0.14.tar.gz
        ${CMAKE_BINARY_DIR}/juce/7.0.14.tar.gz
        STATUS download_status
        TIMEOUT 300
    )
    
    list(GET download_status 0 error_code)
    if(error_code EQUAL 0)
        execute_process(
            COMMAND ${CMAKE_COMMAND} -E tar xf ${CMAKE_BINARY_DIR}/juce/7.0.14.tar.gz
            WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
            RESULT_VARIABLE extract_result
        )
        
        if(NOT extract_result EQUAL 0)
            message(FATAL_ERROR "Failed to extract JUCE archive")
        endif()
        
        # Add JUCE include directory
        include_directories(${CMAKE_BINARY_DIR}/juce/juce_7.0.14/juce/modules)
    else()
        message(FATAL_ERROR "Failed to download JUCE. Please install manually or use: brew install juce")
    endif()
endif()

# Create plugin target
add_library(Vst3Sine SHARED
    src/SineSynth.cpp
    src/SineSynth.h
    src/SineEditor.cpp
    src/SineEditor.h
)

# Platform-specific settings
if(PLATFORM_NAME STREQUAL Mac)
    set_target_properties(Vst3Sine PROPERTIES
        OUTPUT_NAME "Vst3Sine"
        SUFFIX ".plugin"
        MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in
    )

    # macOS code signing
    configure_file(Vst3Sine.entitlements.in ${CMAKE_BINARY_DIR}/Vst3Sine.entitlements @ONLY)
    
    add_custom_command(TARGET Vst3Sine POST_BUILD
        COMMAND codesign
        ARGS --sign "Developer ID Application: Your Name"
             --entitlements ${CMAKE_BINARY_DIR}/Vst3Sine.entitlements
             --force
             ${CMAKE_BINARY_DIR}/Vst3Sine.plugin
        COMMENT "Signing VST3 plugin"
        VERBATIM
    )
    
    target_link_libraries(Vst3Sine
        CoreAudio
        AudioToolbox
    )
    
    # Enable VST3 support
    target_compile_definitions(Vst3Sine PRIVATE
        VST3PLUGIN_API
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_PLUGIN=1
        JUCE_IOS_PLUGIN=0
        JUCE_ANDROID_PLUGIN=0
    )
    
elseif(PLATFORM_NAME STREQUAL Windows)
    set_target_properties(Vst3Sine PROPERTIES
        OUTPUT_NAME "Vst3Sine"
        SUFFIX ".dll"
        PREFIX ""
    )
    
    target_compile_definitions(Vst3Sine PRIVATE
        VST3PLUGIN_API
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_PLUGIN=1
        JUCE_WIN32_GUI=1
    )
    
    # Windows specific
    target_link_libraries(Vst3Sine
        ws2_32
    )
    
elseif(PLATFORM_NAME STREQUAL Linux)
    set_target_properties(Vst3Sine PROPERTIES
        OUTPUT_NAME "Vst3Sine"
        SUFFIX ".component"
    )
    
    target_compile_definitions(Vst3Sine PRIVATE
        VST3PLUGIN_API
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_PLUGIN=1
    )
    
    # Linux specific
    target_link_libraries(Vst3Sine
        pthread
    )
endif()

# JUCE includes
target_include_directories(Vst3Sine PRIVATE
    ${CMAKE_BINARY_DIR}/juce/juce_7.0.14/juce/modules
)

# JUCE modules to include
target_compile_definitions(Vst3Sine PRIVATE
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
    JUCE_VST3_PLUGIN=1
    JUCE_IOS_PLUGIN=0
    JUCE_ANDROID_PLUGIN=0
)

# Tests
enable_testing()

# CppUnit tests
add_executable(SineSynthTests
    tests/SineSynthTests.cpp
)

target_include_directories(SineSynthTests PRIVATE
    ${CMAKE_BINARY_DIR}/juce/juce_7.0.14/juce/modules
    ${CMAKE_SOURCE_DIR}/src
)

target_compile_definitions(SineSynthTests PRIVATE
    JUCE_WEB_BROWSER=0
    JUCE_USE_CURL=0
)

# Link against plugin
target_link_libraries(SineSynthTests PRIVATE Vst3Sine)

# Add to CTest
add_test(NAME SineSynthTests COMMAND SineSynthTests)

add_subdirectory(tests)

# Copy plugin to install directory
install(TARGETS Vst3Sine
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/VST3
)

message(STATUS "Building Vst3Sine plugin for ${PLATFORM_NAME}")
