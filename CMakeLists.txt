cmake_minimum_required(VERSION 3.12)

project(cimgui)

set (CMAKE_CXX_STANDARD 11)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")
set(IMGUI_LIBRARIES )

function(add_cimgui_library libname)

    if (IMGUI_STATIC)
        add_library(${libname} STATIC)
    else (IMGUI_STATIC)
        add_library(${libname} SHARED)
    endif (IMGUI_STATIC)

    target_include_directories(${libname} PUBLIC ${libname})
    set_target_properties(${libname} PROPERTIES PREFIX "")
    install(TARGETS ${libname}
        RUNTIME DESTINATION .
        LIBRARY DESTINATION .
        ARCHIVE DESTINATION .
    )
endfunction()
function(add_cimgui_ext_library libname)
    add_cimgui_library(${libname})
    target_include_directories(${libname} PUBLIC cimgui/imgui)
    target_link_libraries(${libname} PUBLIC cimgui)

    if (WIN32)
        target_compile_definitions(${libname} PRIVATE "IMGUI_API=__declspec(dllimport)")
    endif()
endfunction()

# cimgui
add_cimgui_library(cimgui)
target_sources(cimgui PRIVATE
    cimgui/cimgui.cpp
    cimgui/imgui/imgui.cpp
    cimgui/imgui/imgui_draw.cpp
    cimgui/imgui/imgui_demo.cpp
    cimgui/imgui/imgui_widgets.cpp
)
if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/cimgui/imgui/imgui_tables.cpp)
    target_sources(cimgui PRIVATE cimgui/imgui/imgui_tables.cpp)
endif()
if (WIN32)
    target_compile_definitions(cimgui PRIVATE "IMGUI_API=__declspec(dllexport)")
    target_link_libraries(cimgui PUBLIC imm32)
endif()
if (IMGUI_FREETYPE)
	FIND_PACKAGE(freetype REQUIRED PATHS ${FREETYPE_PATH})
	target_link_libraries(cimgui PUBLIC freetype)
	target_sources(cimgui PRIVATE cimgui/imgui/misc/freetype/imgui_freetype.cpp)
	add_definitions("-DCIMGUI_FREETYPE=1")
endif (IMGUI_FREETYPE)

# cimguizmo
add_cimgui_ext_library(cimguizmo)
target_sources(cimguizmo PRIVATE
    cimguizmo/cimguizmo.cpp
    cimguizmo/ImGuizmo/ImGuizmo.cpp
)