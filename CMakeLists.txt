cmake_minimum_required(VERSION 2.6)

project(emulationstation)

#-------------------------------------------------------------------------------
#add local find scripts to CMAKE path
LIST(APPEND CMAKE_MODULE_PATH
    ${CMAKE_CURRENT_SOURCE_DIR}/CMake/Utils
    ${CMAKE_CURRENT_SOURCE_DIR}/CMake/Packages
)

#-------------------------------------------------------------------------------
#set up OpenGL system variable
set(GLSystem "Desktop OpenGL" CACHE STRING "The OpenGL system to be used")
set_property(CACHE GLSystem PROPERTY STRINGS "Desktop OpenGL" "OpenGL ES")

#-------------------------------------------------------------------------------
#check if we're running on Raspberry Pi
MESSAGE("Looking for bcm_host.h")
if(EXISTS "/opt/vc/include/bcm_host.h")
    MESSAGE("bcm_host.h found")
    set(BCMHOST found)
    set(GLSystem "OpenGL ES")
else()
    MESSAGE("bcm_host.h not found")
endif()

#finding necessary packages
#-------------------------------------------------------------------------------
if(${GLSystem} MATCHES "Desktop OpenGL")
    find_package(OpenGL REQUIRED)
else()
    find_package(OpenGLES REQUIRED)
endif()
find_package(FreeType REQUIRED)
find_package(FreeImage REQUIRED)
find_package(SDL REQUIRED)
find_package(Boost REQUIRED)

#-------------------------------------------------------------------------------
#set up compiler flags and excutable names
if(DEFINED BCMHOST)
    add_definitions(-D_RPI_)
endif()

if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS "-std=c++11") #support C++11
    set(CMAKE_CXX_FLAGS "-O2")        #optimize
    set(CMAKE_EXE_LINKER_FLAGS "-s")  #strip binary
endif()

if(${GLSystem} MATCHES "Desktop OpenGL")
    add_definitions(-DUSE_OPENGL_DESKTOP)
else()
    add_definitions(-DUSE_OPENGL_ES)
endif()

#-------------------------------------------------------------------------------
#add include directories
set(ES_INCLUDE_DIRS
    ${FREETYPE_INCLUDE_DIRS}
    ${FreeImage_INCLUDE_DIRS}
    ${SDL_INCLUDE_DIR}
    ${Boost_INCLUDE_DIRS}
)

if(${GLSystem} MATCHES "Desktop OpenGL")
    LIST(APPEND ES_INCLUDE_DIRS
        ${OPENGL_INCLUDE_DIR}
    )
else()
    LIST(APPEND ES_INCLUDE_DIRS
        ${OPENGLES_INCLUDE_DIR}
    )
endif()   

if(DEFINED BCMHOST)
    LIST(APPEND ES_INCLUDE_DIRS
        "/opt/vc/include"
        "/opt/vc/include/interface/vcos"
        "/opt/vc/include/interface/vmcs_host/linux"
        "/opt/vc/include/interface/vcos/pthreads"
    )
endif()

#-------------------------------------------------------------------------------
#define basic sources and headers
set(ES_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/AudioManager.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/FileData.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/FolderData.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Font.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/GameData.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Gui.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/InputConfig.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/InputManager.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Log.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/MathExp.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/platform.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Renderer.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Sound.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/SystemData.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/XMLReader.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiAnimation.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiBox.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiDetectDevice.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiFastSelect.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiGameList.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiImage.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiInputConfig.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiList.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiMenu.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiTheme.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/pugiXML/pugiconfig.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/pugiXML/pugixml.hpp
)
set(ES_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/src/AudioManager.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/FolderData.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Font.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/GameData.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Gui.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/InputConfig.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/InputManager.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Log.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/MathExp.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/platform.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Renderer_draw_gl.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Renderer_init.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Sound.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/SystemData.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/XMLReader.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiAnimation.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiBox.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiDetectDevice.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiFastSelect.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiGameList.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiImage.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiInputConfig.cpp
#    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiList.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiMenu.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/components/GuiTheme.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/pugiXML/pugixml.cpp
)

#add OS specifix sources
#if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
#    LIST(APPEND ES_SOURCES
#        main_windows.cpp
#    )
#endif()
#if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
#    LIST(APPEND ES_SOURCES
#        main_linux.cpp
#    )
#endif()

#add open gl specific sources
if(${GLSystem} MATCHES "Desktop OpenGL")
    LIST(APPEND ES_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/src/Renderer_init_sdlgl.cpp
    )
else()
    LIST(APPEND ES_SOURCES
		${CMAKE_CURRENT_SOURCE_DIR}/src/Renderer_init_gles.cpp
    )
endif()

#-------------------------------------------------------------------------------
#define libraries and directories
set(ES_LIBRARIES
#    ${FREETYPE_LIBRARIES}
#    ${FreeImage_LIBRARIES}
#    ${SDL_LIBRARY}
#    ${Boost_LIBRARIES}
	freetype
	freeimage
	SDL
	boost_filesystem
	boost_system
)

if(${GLSystem} MATCHES "Desktop OpenGL")
    LIST(APPEND ES_LIBRARIES
        ${OPENGL_LIBRARIES}
    )
else()
    LIST(APPEND ES_LIBRARIES
        ${OPENGLES_LIBRARIES}
    )
endif()

if(DEFINED BCMHOST)
    LIST(APPEND ES_LIBRARIES
        bcm_host
    )
endif()

if(DEFINED BCMHOST)
    LINK_DIRECTORIES(${CMAKE_CURRENT_BINARY_DIR} "/opt/vc/lib")
endif()

#-------------------------------------------------------------------------------
#set up build directories
set(dir ${CMAKE_CURRENT_SOURCE_DIR})
set(EXECUTABLE_OUTPUT_PATH ${dir} CACHE PATH "Build directory" FORCE)
set(LIBRARY_OUTPUT_PATH ${dir} CACHE PATH "Build directory" FORCE)

#-------------------------------------------------------------------------------
#define target
include_directories(${ES_INCLUDE_DIRS})
add_executable(emulationstation ${ES_SOURCES} ${ES_HEADERS})
target_link_libraries(emulationstation ${ES_LIBRARIES})