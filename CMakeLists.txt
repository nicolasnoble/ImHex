cmake_minimum_required(VERSION 3.16)

project(HexEditor)

SET(CMAKE_FIND_PACKAGE_SORT_ORDER NATURAL)
SET(CMAKE_FIND_PACKAGE_SORT_DIRECTION DEC)
set(CMAKE_CXX_STANDARD 20)

# Enforce that we use non system Python 3 on macOS.
set(Python_FIND_FRAMEWORK NEVER)


find_package(PkgConfig REQUIRED)
pkg_search_module(GLFW REQUIRED glfw3)
pkg_search_module(GLM REQUIRED glm)
pkg_search_module(CRYPTO REQUIRED libcrypto)
pkg_search_module(CAPSTONE REQUIRED capstone)
find_package(OpenGL REQUIRED)
find_package(nlohmann_json REQUIRED)
find_package(Freetype REQUIRED)
find_package(Python COMPONENTS Interpreter Development)

add_subdirectory(external/llvm/Demangle)

if(Python_VERSION LESS 3)
    message(STATUS ${PYTHON_VERSION_MAJOR_MINOR})
    message(FATAL_ERROR "No valid version of Python 3 was found.")
endif()

pkg_search_module(MAGIC libmagic)
if(NOT MAGIC_FOUND)
    find_library(MAGIC magic REQUIRED)
endif()

include_directories(include ${GLFW_INCLUDE_DIRS} ${GLM_INCLUDE_DIRS} ${CRYPTO_INCLUDE_DIRS} ${CAPSTONE_INCLUDE_DIRS} ${FREETYPE_INCLUDE_DIRS} ${MAGIC_INCLUDE_DIRS} libs/ImGui/include libs/glad/include ${Python_INCLUDE_DIRS})

# Get Python major and minor
string(REPLACE "." ";" PYTHON_VERSION_MAJOR_MINOR ${Python_VERSION})
list(REMOVE_AT PYTHON_VERSION_MAJOR_MINOR 2)
list(JOIN PYTHON_VERSION_MAJOR_MINOR "." PYTHON_VERSION_MAJOR_MINOR)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_C_FLAGS} -DIMGUI_IMPL_OPENGL_LOADER_GLAD -DPYTHON_VERSION_MAJOR_MINOR=\"\\\"${PYTHON_VERSION_MAJOR_MINOR}\"\\\"")

if (WIN32)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static-libstdc++ -static-libgcc -static")
    SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wl,-subsystem,windows")
endif (WIN32)

SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -DRELEASE")
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -DDEBUG")

add_executable(ImHex
        source/main.cpp
        source/window.cpp

        source/helpers/utils.cpp
        source/helpers/crypto.cpp
        source/helpers/patches.cpp
        source/helpers/math_evaluator.cpp
        source/helpers/project_file_handler.cpp
        source/helpers/loader_script_handler.cpp

        source/lang/preprocessor.cpp
        source/lang/lexer.cpp
        source/lang/parser.cpp
        source/lang/validator.cpp
        source/lang/evaluator.cpp

        source/providers/file_provider.cpp

        source/views/view_hexeditor.cpp
        source/views/view_pattern.cpp
        source/views/view_pattern_data.cpp
        source/views/view_hashes.cpp
        source/views/view_information.cpp
        source/views/view_help.cpp
        source/views/view_tools.cpp
        source/views/view_strings.cpp
        source/views/view_data_inspector.cpp
        source/views/view_disassembler.cpp
        source/views/view_bookmarks.cpp
        source/views/view_patches.cpp
        source/views/view_command_palette.cpp

        libs/glad/source/glad.c

        libs/ImGui/source/imgui.cpp
        libs/ImGui/source/imgui_draw.cpp
        libs/ImGui/source/imgui_widgets.cpp
        libs/ImGui/source/imgui_demo.cpp
        libs/ImGui/source/imgui_freetype.cpp
        libs/ImGui/source/imgui_impl_glfw.cpp
        libs/ImGui/source/imgui_impl_opengl3.cpp
        libs/ImGui/source/ImGuiFileBrowser.cpp
        libs/ImGui/source/TextEditor.cpp

        resource.rc
        )

set_target_properties(ImHex PROPERTIES CXX_VISIBILITY_PRESET hidden)

target_link_directories(ImHex PRIVATE ${GLFW_LIBRARY_DIRS} ${CRYPTO_LIBRARY_DIRS} ${CAPSTONE_LIBRARY_DIRS} ${MAGIC_LIBRARY_DIRS})

if (WIN32)
    target_link_libraries(ImHex libglfw3.a libgcc.a libstdc++.a libmagic.a libgnurx.a libtre.a libintl.a libiconv.a shlwapi.lib libcrypto.a libwinpthread.a libcapstone.a LLVMDemangle ${Python_LIBRARIES} nlohmann_json::nlohmann_json Freetype::Freetype)
elseif (UNIX)
    target_link_libraries(ImHex glfw magic crypto ${CMAKE_DL_LIBS} capstone LLVMDemangle ${Python_LIBRARIES} nlohmann_json::nlohmann_json Freetype::Freetype)
endif()

install(TARGETS ImHex DESTINATION bin)
