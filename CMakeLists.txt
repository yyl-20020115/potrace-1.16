cmake_minimum_required(VERSION 3.16)

project(potrace VERSION 1.16 LANGUAGES C)

include(GNUInstallDirs)
include(CheckFunctionExists)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

check_function_exists(getopt_long HAVE_GETOPT_LONG)

add_library(potrace_lib
    src/curve.c
    src/trace.c
    src/decompose.c
    src/potracelib.c
)

target_include_directories(potrace_lib
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)


set(potrace_sources
    src/main.c
    src/bitmap_io.c
    src/backend_eps.c
    src/backend_pgm.c
    src/backend_svg.c
    src/backend_xfig.c
    src/backend_dxf.c
    src/backend_pdf.c
    src/backend_geojson.c
    src/flate.c
    src/lzw.c
    src/greymap.c
    src/render.c
    src/bbox.c
    src/trans.c
    src/progress_bar.c
)

if(NOT HAVE_GETOPT_LONG)
    list(APPEND potrace_sources
        src/getopt.c
        src/getopt1.c
    )
endif()

add_executable(potrace ${potrace_sources})

target_include_directories(potrace
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/include/getopt
)

set(mkbitmap_sources
    src/mkbitmap.c
    src/bitmap_io.c
    src/greymap.c
)

if(NOT HAVE_GETOPT_LONG)
    list(APPEND mkbitmap_sources
        src/getopt.c
        src/getopt1.c
    )
endif()

add_executable(mkbitmap ${mkbitmap_sources})

target_include_directories(mkbitmap
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/include/getopt
)
add_executable(potracelib_demo src/potracelib_demo.c)
target_include_directories(potracelib_demo PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)

install(TARGETS potrace mkbitmap potracelib_demo
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(TARGETS potrace_lib
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
else()
target_link_libraries(mkbitmap PRIVATE m)
target_link_libraries(potrace_lib PUBLIC m)
target_link_libraries(potracelib_demo PRIVATE m)
target_link_libraries(potrace PRIVATE m)
endif()

target_link_libraries(potrace PRIVATE potrace_lib)
target_link_libraries(potracelib_demo PRIVATE potrace_lib)

install(FILES src/potracelib.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
