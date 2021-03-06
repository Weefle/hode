project(hode)

cmake_minimum_required(VERSION 3.0)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

#include(FindPkgConfig)


set(CMAKE_CXX_FLAGS "-g -Wall -pedantic -MMD ${CMAKE_CXX_FLAGS}")

#pkg_check_modules(SDL2 sdl2 REQUIRED)
find_package(SDL2 REQUIRED)

include_directories(${SDL2_INCLUDE_DIR})

if (VITA)
    set(ENV{PKG_CONFIG_PATH} "$ENV{VITASDK}/arm-vita-eabi/lib/pkgconfig")
endif (VITA)

file(GLOB SRC *.cpp 3p/inih/ini.c 3p/libxbr-standalone/xbr.c)
list(FILTER SRC EXCLUDE REGEX ".*android.cpp|system_psp.cpp|system_wii.cpp")
add_executable(${CMAKE_PROJECT_NAME}
        ${SRC}
        )

target_link_libraries(${CMAKE_PROJECT_NAME} -lmingw32 -lSDL2main -lSDL2 -mwindows)
#target_link_libraries(${CMAKE_PROJECT_NAME} ${SDL2_LIBRARY} -lmingw32 -lSDL2main -lSDL2 -mwindows)

if (NSWITCH)
    add_definitions(-D__SWITCH__)
    add_custom_target(${CMAKE_PROJECT_NAME}.nro
            DEPENDS ${CMAKE_PROJECT_NAME}
            COMMAND nacptool --create "Heart of Darkness" "cyx, usineur" "0.2.9d" ${CMAKE_PROJECT_NAME}.nacp
            COMMAND elf2nro ${CMAKE_PROJECT_NAME} ${CMAKE_PROJECT_NAME}.nro --icon=${CMAKE_SOURCE_DIR}/3p/res/icon.jpg --nacp=${CMAKE_PROJECT_NAME}.nacp
            )
    add_custom_target(nxlink
            COMMAND nxlink -a $(SWITCHIP) ${CMAKE_PROJECT_NAME}.nro -s -p ${CMAKE_PROJECT_NAME}/${CMAKE_PROJECT_NAME}.nro
            DEPENDS ${CMAKE_PROJECT_NAME}.nro
            )
endif (NSWITCH)

if (VITA)
    include("${VITASDK}/share/vita.cmake" REQUIRED)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -fpermissive")
    vita_create_self(eboot.bin ${CMAKE_PROJECT_NAME} UNSAFE)
    vita_create_vpk(${CMAKE_PROJECT_NAME}.vpk "HODEVITA1" eboot.bin
            NAME ${CMAKE_PROJECT_NAME}
            FILE 3p/res/icon0.png sce_sys/icon0.png
            FILE 3p/res/bg.png sce_sys/livearea/contents/bg.png
            FILE 3p/res/startup.png sce_sys/livearea/contents/startup.png
            FILE 3p/res/template.xml sce_sys/livearea/contents/template.xml
            )
endif (VITA)
