cmake_minimum_required(VERSION 3.15)
project(ShowMIDI VERSION 0.6.2)

# To build a CLAP you need to do the following in addition to editing this file
#
# - First do the native build of at least the ShowMIDI_shared target on your platform as Debug or Release
# - Then do `cmake -Bbuild_clap -DCMAKE_BUILD_TYPE=Debug` or Release of course
# - Then do `cmake --build build_clap --config Debug` or Release
# 
# This will result in build_clap/ShowMIDI_artefacts/Debug/ShowMIDI.clap (or Release).
#
# For this to work a couple of things need to be configured

# Next adjust these paths to point to your JUCE and the current checkout of ClapJuce Extensions
include(./cmake/CPM.cmake)

CPMAddPackage("gh:juce-framework/JUCE#7.0.8")

# Make sure to set the same C++ version as you have set in the Projucer
set(CMAKE_CXX_STANDARD 17)

# Make sure to set the same MacOS deployment target as you have set in the Projucer
set(CMAKE_OSX_DEPLOYMENT_TARGET "11.7" CACHE STRING "Minimum OS X deployment target")

# If the Projucer is using "static runtime" for Visual Studio:
#set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>" CACHE STRING "Runtime")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded" CACHE STRING "Runtime")

# This bypasses XCode signing and is appropriate if you sign after a build. Comment it out if you want to use xcode signing.
set(CMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_REQUIRED "NO")
set(CMAKE_XCODE_ATTRIBUTE_CODE_SIGNING_ALLOWED "NO")

set(COMPANY_NAME "Uwyn")
set(COMPANY_WEBSITE "https://uwyn.com/")
set(BUNDLE_ID "com.uwyn.showmidi.clap")
set(ICON_BIG "${CMAKE_CURRENT_SOURCE_DIR}/Assets/appicon.png")

juce_add_plugin("${CMAKE_PROJECT_NAME}"
        ICON_BIG ${ICON_BIG}
        ICON_SMALL ${ICON_BIG}
        COMPANY_NAME ${COMPANY_NAME}
        COMPANY_WEBSITE ${COMPANY_WEBSITE}
        COMPANY_EMAIL ${COMPANY_EMAIL}
        BUNDLE_ID ${BUNDLE_ID}
        IS_SYNTH FALSE
        NEEDS_MIDI_INPUT TRUE
        NEEDS_MIDI_OUTPUT TRUE
        IS_MIDI_EFFECT FALSE
        EDITOR_WANTS_KEYBOARD_FOCUS FALSE
        COPY_PLUGIN_AFTER_BUILD FALSE
        PLUGIN_NAME "${CMAKE_PROJECT_NAME}"
        PLUGIN_MANUFACTURER_CODE Manu
        PLUGIN_CODE Qyjy
        FORMATS AU VST3 VST2 Standalone LV2
        PRODUCT_NAME "${CMAKE_PROJECT_NAME}"
        MICROPHONE_PERMISSION_ENABLED TRUE
        BLUETOOTH_PERMISSION_ENABLED TRUE
        APP_SANDBOX_ENABLED FALSE
        HARDENED_RUNTIME_ENABLED TRUE
        HARDENED_RUNTIME_OPTIONS
        com.apple.security.cs.allow-jit
        com.apple.security.cs.allow-unsigned-executable-memory
        com.apple.security.cs.disable-library-validation
        com.apple.security.device.audio-input
        EDITOR_NEEDS_KEYBOARD_FOCUS TRUE
)

CPMAddPackage(NAME clap-juce-extensions
        GITHUB_REPOSITORY "multivac61/clap-juce-extensions"
        GIT_TAG "migrate-from-git-submodules-to-cpm"
        OPTIONS "CLAP_JUCE_EXTENSIONS_BUILD_EXAMPLES OFF")

include(${clap-juce-extensions_SOURCE_DIR}/cmake/JucerClap.cmake)

clap_juce_extensions_plugin(
        TARGET ${PROJECT_NAME}
        PLUGIN_CODE shmi
        CLAP_ID "${BUNDLE_ID}"
        CLAP_FEATURES note-effect utility
)

juce_add_binary_data(${CMAKE_PROJECT_NAME}_BinaryData
        SOURCES
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/appicon.png
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/collapsed.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/expanded.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/help.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/hidden.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/settings.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Assets/visible.svg
        ${CMAKE_CURRENT_SOURCE_DIR}/Fonts/JetBrainsMono-Italic.ttf
        ${CMAKE_CURRENT_SOURCE_DIR}/Fonts/JetBrainsMono-Regular.ttf
        ${CMAKE_CURRENT_SOURCE_DIR}/Fonts/JetBrainsMono-SemiBold.ttf
        ${CMAKE_CURRENT_SOURCE_DIR}/Fonts/JetBrainsMono-SemiBoldItalic.ttf
)

juce_generate_juce_header(${CMAKE_PROJECT_NAME})
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/AboutComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/AboutComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/Main.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/MainLayoutComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/MidiDeviceComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/MidiDevicesListener.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PaintedButton.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PluginEditor.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PluginProcessor.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PluginSettings.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PopupColourSelector.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PortListComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/PropertiesSettings.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/SettingsComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/ShowMidiApplication.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/SidebarComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/StandaloneDevicesComponent.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/StandaloneWindow.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/Theme.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/Source/UwynLookAndFeel.cpp
)

add_dependencies(${CMAKE_PROJECT_NAME} ${CMAKE_PROJECT_NAME}_BinaryData)

target_compile_definitions(${CMAKE_PROJECT_NAME}
        PUBLIC
        JUCE_WEB_BROWSER=0
        JUCE_USE_CURL=0
        JUCE_VST3_CAN_REPLACE_VST2=0
        JUCE_USE_CUSTOM_PLUGIN_STANDALONE_APP=1
        JUCE_MODAL_LOOPS_PERMITTED=1

        JUCE_PLUGINHOST_LADSPA=1
        JUCE_PLUGINHOST_VST=0
        JUCE_PLUGINHOST_VST3=1
        JUCE_PLUGINHOST_AU=1

        $<$<CXX_COMPILER_ID:MSVC>:
        JUCE_USE_WINRT_MIDI=0
        JUCE_FORCE_WINRT_MIDI=0
        >

        $<$<CXX_COMPILER_ID:AppleClang>:JUCE_SILENCE_XCODE_15_LINKER_WARNING=1>
)

target_include_directories(${CMAKE_PROJECT_NAME}
        PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/source
)

target_compile_options(${CMAKE_PROJECT_NAME}
        PUBLIC
        $<$<CXX_COMPILER_ID:AppleClang>:
        -Wall
        -Wshadow-all
        -Wshorten-64-to-32
        -Wstrict-aliasing
        -Wuninitialized
        -Wunused-parameter
        -Wconversion
        -Wsign-compare
        -Wint-conversion
        -Wconditional-uninitialized
        -Woverloaded-virtual
        -Wreorder
        -Wconstant-conversion
        -Wsign-conversion
        -Wunused-private-field
        -Wbool-conversion
        -Wextra-semi
        -Wunreachable-code
        -Wzero-as-null-pointer-constant
        -Wcast-align
        -Winconsistent-missing-destructor-override
        -Wshift-sign-overflow
        -Wnullable-to-nonnull-conversion
        -Wunused-variable
        -Wpedantic
        -Wno-c99-extensions
        >
)

target_link_libraries(${CMAKE_PROJECT_NAME}
        PRIVATE
        juce::juce_audio_utils
        juce::juce_audio_devices
        juce::juce_audio_plugin_client
        juce::juce_audio_processors
        juce::juce_recommended_config_flags
        juce::juce_recommended_lto_flags

        ${CMAKE_PROJECT_NAME}_BinaryData
)

if (APPLE)
    target_link_options(${CMAKE_PROJECT_NAME} PUBLIC $<$<CXX_COMPILER_ID:AppleClang>:-Wl,-ld_classic>)
endif ()