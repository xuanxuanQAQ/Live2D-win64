# CMakeList.txt: Live2D 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。
#
cmake_minimum_required (VERSION 3.8)

# Enable Hot Reload for MSVC compilers if supported.
if (POLICY CMP0141)
  cmake_policy(SET CMP0141 NEW)
  set(CMAKE_MSVC_DEBUG_INFORMATION_FORMAT "$<IF:$<AND:$<C_COMPILER_ID:MSVC>,$<CXX_COMPILER_ID:MSVC>>,$<$<CONFIG:Debug,RelWithDebInfo>:EditAndContinue>,$<$<CONFIG:Debug,RelWithDebInfo>:ProgramDatabase>>")
endif()


project ("Live2D")

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
cmake_policy(SET CMP0079 NEW)

set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
set(BUILD_UTILS OFF CACHE BOOL "" FORCE)

add_library(Live2DCubismCore STATIC IMPORTED)
set(CRT MT)
set_target_properties(Live2DCubismCore
  PROPERTIES
    IMPORTED_LOCATION_DEBUG
    ${CMAKE_CURRENT_SOURCE_DIR}/third/live2d/lib/Live2DCubismCore_${CRT}d.lib
    IMPORTED_LOCATION_RELEASE
    ${CMAKE_CURRENT_SOURCE_DIR}/third/live2d/lib/Live2DCubismCore_${CRT}.lib
    INTERFACE_INCLUDE_DIRECTORIES
    ${CMAKE_CURRENT_SOURCE_DIR}/third/live2d/inc/
)

set(GLEW_PATH third/glew)
set(GLFW_PATH third/glfw)
add_subdirectory(${GLEW_PATH}/build/cmake)
add_subdirectory(${GLFW_PATH})
set(FRAMEWORK_SOURCE OpenGL)
add_subdirectory(Framework)
target_compile_definitions(Framework PUBLIC CSM_TARGET_WIN_GL)
target_include_directories(Framework PUBLIC ${GLEW_PATH}/include ${GLFW_PATH}/include)
target_link_libraries(Framework Live2DCubismCore glew_s ${OPENGL_LIBRARIES})

find_package(OpenGL REQUIRED)

include_directories(include)
include_directories(src)
include_directories(third/stb)

aux_source_directory(src Live2D_SOURCES_FILE)
add_executable (Live2D ${Live2D_SOURCES_FILE})

set_target_properties(Live2D PROPERTIES LINK_FLAGS "/SUBSYSTEM:CONSOLE")
target_link_libraries(Live2D PRIVATE Live2DCubismCore Framework glfw ${OPENGL_LIBRARIES} debug -NODEFAULTLIB:libcmtd.lib optimized -NODEFAULTLIB:libcmt.lib)

if (CMAKE_VERSION VERSION_GREATER 3.12)
  set_property(TARGET Live2D PROPERTY CXX_STANDARD 20)
endif()

# TODO: 如有需要，请添加测试并安装目标。
