cmake_minimum_required (VERSION 2.6)
project (RadeonGPUAnalyzerBackend)

# The version number
set (RadeonGPUAnalyzerBackend_VERSION_MAJOR 1)
set (RadeonGPUAnalyzerBackend_VERSION_MINOR 0)

# Project-specific preprocessor directives
add_definitions(-DRGA_BACKEND_EXPORTS)

# Windows-specific preprocessor directives
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Add these definitions if static linkage is used with the other projects
add_definitions(-DAMDTBASETOOLS_STATIC -DAMDTOSWRAPPERS_STATIC -DAMDTOSAPIWRAPPERS_STATIC)
 
# Configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/RadeonGPUAnalyzerBackendConfig.h.in"
  "${PROJECT_BINARY_DIR}/RadeonGPUAnalyzerBackendConfig.h"
  ) 

# Include directories
include_directories("${PROJECT_SOURCE_DIR}/../")
include_directories("${PROJECT_SOURCE_DIR}/include")
include_directories("${PROJECT_SOURCE_DIR}/include/Common")
include_directories("${PROJECT_SOURCE_DIR}/emulator/Parser")
include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/ADLUtil")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/CElf/Include")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/DeviceInfo")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/AMDTMutex")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/TSingleton")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/Misc")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/ACLModuleManager")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/ADL/include")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/DX10ASM")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/CAL/8.95")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/CAL/8.95/include")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/ACL/TOT/include")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/AMD/APPSDK/3-0/include")
include_directories("${PROJECT_SOURCE_DIR}/../../Common/Lib/Ext/Boost/boost_1_59_0")

    
# Windows-specific include files
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    include_directories("${PROJECT_SOURCE_DIR}/../Core/DX10ASM/Include")
    include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils")
    include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX10")
    include_directories("${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX11")
endif()


# Source files that are common to Windows and Linux
file(GLOB COMMON_SRC
    "src/*.cpp"
    "include/*.h"
    "emulator/Parser/Instruction.cpp"
    "emulator/Parser/ISAParser.cpp"
    "emulator/Parser/ISAProgramGraph.cpp"
    "emulator/Parser/ParserSI.cpp"
    "emulator/Parser/ParserSIDS.cpp"
    "emulator/Parser/ParserSIEXP.cpp"
    "emulator/Parser/ParserSIMIMG.cpp"
    "emulator/Parser/ParserSIMTBUF.cpp"
    "emulator/Parser/ParserSIMUBUF.cpp"
    "emulator/Parser/ParserSISMRD.cpp"
    "emulator/Parser/ParserSISOP1.cpp"
    "emulator/Parser/ParserSISOP2.cpp"
    "emulator/Parser/ParserSISOPC.cpp"
    "emulator/Parser/ParserSISOPK.cpp"
    "emulator/Parser/ParserSISOPP.cpp"
    "emulator/Parser/ParserSIVINTRP.cpp"
    "emulator/Parser/ParserSIVOP.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule/DynamicLibraryModule.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule/ACLModule.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ACLModuleManager/ACLModuleManager.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ADLUtil/ADLUtil.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/AMDTMutex/AMDTMutex.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule/CALModule.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/CElf/Src/CElf.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/CElf/Src/CElfSection.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/CElf/Src/CElfStringTable.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/CElf/Src/CElfSymbolTable.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DeviceInfo/DeviceInfoUtils.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/Misc/GDT_Memory.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule/OpenCLModule.cpp"
)

# Windows-specific source files
file(GLOB WINDOWS_SRC
    "${PROJECT_SOURCE_DIR}/../../Common/Src/DynamicLibraryModule/DXXModule.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX10/D3D10ShaderObject.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX10/D3D10ShaderUtils.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX10/DXBCChecksum.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/DX10/IBlob.cpp"
    "${PROJECT_SOURCE_DIR}/../../Common/Src/ShaderUtils/StringUtils.cpp"
)

# Linux-specific source files
file(GLOB LINUX_SRC
    "src/linux/*.h"
    "src/linux/*.cpp"
)

# Pick up the source files that are relevant to the platform
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    add_library(RadeonGPUAnalyzerBackend STATIC ${COMMON_SRC} ${WINDOWS_SRC})
    if (CMAKE_BUILD_TYPE STREQUAL "Release")
        # Release Build
        if (CMAKE_64BIT_TARGET)
            find_library(DX10ASM_LIBRARY_RELEASE DX10ASMLib_s HINTS "${PROJECT_SOURCE_DIR}/../Core/DX10ASM/Lib/VS2015/x64/Release_Static")
            target_link_libraries (RadeonGPUAnalyzerBackend ${DX10ASM_LIBRARY_RELEASE})  
        else()
            find_library(DX10ASM_LIBRARY_RELEASE DX10ASMLib_s HINTS "${PROJECT_SOURCE_DIR}/../Core/DX10ASM/Lib/VS2015/Win32/Release_Static")
            target_link_libraries (RadeonGPUAnalyzerBackend ${DX10ASM_LIBRARY_RELEASE})  
        endif()
    else()
        # Debug build
        if (CMAKE_64BIT_TARGET)
            find_library(DX10ASM_LIBRARY_DEBUG DX10ASMLib_s HINTS "${PROJECT_SOURCE_DIR}/../Core/DX10ASM/Lib/VS2015/x64/Debug_Static")
            target_link_libraries (RadeonGPUAnalyzerBackend ${DX10ASM_LIBRARY_DEBUG})
        else()
            find_library(DX10ASM_LIBRARY_DEBUG DX10ASMLib_s HINTS "${PROJECT_SOURCE_DIR}/../Core/DX10ASM/Lib/VS2015/Win32/Debug_Static")
            target_link_libraries (RadeonGPUAnalyzerBackend ${DX10ASM_LIBRARY_DEBUG})
        endif()
    endif()
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    add_library(RadeonGPUAnalyzerBackend STATIC ${COMMON_SRC} ${LINUX_SRC})
endif()

# set binary suffix
set_target_properties(RadeonGPUAnalyzerBackend PROPERTIES DEBUG_POSTFIX -d)

# Link settings
if (CMAKE_64BIT_TARGET)
    link_directories(${PROJECT_SOURCE_DIR}/../../Lib/Ext/zlib/1.2.8/contrib/vstudio/vc14/x64/ZlibStatRelease)
else()
    link_directories(${PROJECT_SOURCE_DIR}/../../Lib/Ext/zlib/1.2.8/contrib/vstudio/vc14/x86/ZlibStatRelease)
endif()
target_link_libraries (RadeonGPUAnalyzerBackend AMDTOSWrappers AMDTBaseTools AMDTOSAPIWrappers)

