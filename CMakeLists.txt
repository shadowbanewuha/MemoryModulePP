# Put this file at: third_party/MemoryModulePP/CMakeLists.txt
# Directory layout expected:
#   third_party/MemoryModulePP/
#     CMakeLists.txt
#     MemoryModule/
#     3rdparty/

cmake_minimum_required(VERSION 3.15)

add_library(MemoryModulePP STATIC
    3rdparty/Detours/creatwth.cpp
    3rdparty/Detours/detours.cpp
    3rdparty/Detours/disasm.cpp
    3rdparty/Detours/disolarm.cpp
    3rdparty/Detours/disolarm64.cpp
    3rdparty/Detours/disolia64.cpp
    3rdparty/Detours/disolx64.cpp
    3rdparty/Detours/disolx86.cpp
    3rdparty/Detours/image.cpp
    3rdparty/Detours/modules.cpp

    MemoryModule/BaseAddressIndex.cpp
    MemoryModule/ImportTable.cpp
    MemoryModule/Initialize.cpp
    MemoryModule/InvertedFunctionTable.cpp
    MemoryModule/LdrEntry.cpp
    MemoryModule/LoadDllMemoryApi.cpp
    MemoryModule/Loader.cpp
    MemoryModule/MemoryModule.cpp
    MemoryModule/MmpDotNet.cpp
    MemoryModule/MmpLdrpTls.cpp
    MemoryModule/MmpTls.cpp
    MemoryModule/MmpTlsFiber.cpp
    MemoryModule/ReflectiveLoader.c
    MemoryModule/Utils.cpp
)

add_library(MemoryModulePP::MemoryModulePP ALIAS MemoryModulePP)

target_include_directories(MemoryModulePP
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/MemoryModule
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/Detours
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/phnt/include
)

target_compile_definitions(MemoryModulePP
    PUBLIC
        UNICODE
        _UNICODE
    PRIVATE
        _MEMORY_MODULE
        _HAS_AUTO_INITIALIZE
        _LIB
)

# Detours chooses architecture by these macros. MSBuild usually provides them,
# but CMake+MSVC may not always do so explicitly, so define them here.
if(MSVC)
    if(CMAKE_SIZEOF_VOID_P EQUAL 8)
        if(CMAKE_SYSTEM_PROCESSOR MATCHES "ARM64|aarch64")
            target_compile_definitions(MemoryModulePP PRIVATE _ARM64_)
        else()
            target_compile_definitions(MemoryModulePP PRIVATE _AMD64_)
        endif()
    else()
        if(CMAKE_SYSTEM_PROCESSOR MATCHES "ARM|arm")
            target_compile_definitions(MemoryModulePP PRIVATE _ARM_)
        else()
            target_compile_definitions(MemoryModulePP PRIVATE _X86_)
        endif()
    endif()
endif()

# Keep runtime library consistent with vcpkg x64-windows: Debug=/MDd, Release=/MD.
# This is important if your main project and vcpkg dependencies are all dynamic CRT.
if(MSVC)
    target_compile_options(MemoryModulePP PRIVATE
        $<$<CONFIG:Debug>:/MDd>
        $<$<NOT:$<CONFIG:Debug>>:/MD>
    )
endif()

# The original vcxproj disables /GS for ReflectiveLoader.c.
# Keep the same setting to avoid changing reflective-loader behavior.
if(MSVC)
    set_source_files_properties(MemoryModule/ReflectiveLoader.c PROPERTIES
        COMPILE_OPTIONS "/GS-;/JMC-"
    )
endif()

# Organize files nicely when using Visual Studio generator / MSBuild.
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES
    3rdparty/Detours/creatwth.cpp
    3rdparty/Detours/detours.cpp
    3rdparty/Detours/disasm.cpp
    3rdparty/Detours/disolarm.cpp
    3rdparty/Detours/disolarm64.cpp
    3rdparty/Detours/disolia64.cpp
    3rdparty/Detours/disolx64.cpp
    3rdparty/Detours/disolx86.cpp
    3rdparty/Detours/image.cpp
    3rdparty/Detours/modules.cpp

    MemoryModule/BaseAddressIndex.cpp
    MemoryModule/ImportTable.cpp
    MemoryModule/Initialize.cpp
    MemoryModule/InvertedFunctionTable.cpp
    MemoryModule/LdrEntry.cpp
    MemoryModule/LoadDllMemoryApi.cpp
    MemoryModule/Loader.cpp
    MemoryModule/MemoryModule.cpp
    MemoryModule/MmpDotNet.cpp
    MemoryModule/MmpLdrpTls.cpp
    MemoryModule/MmpTls.cpp
    MemoryModule/MmpTlsFiber.cpp
    MemoryModule/ReflectiveLoader.c
    MemoryModule/Utils.cpp
)
