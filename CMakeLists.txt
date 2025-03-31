cmake_minimum_required(VERSION 3.14...3.31)
project(pngcheck
        VERSION 4.0.0
        DESCRIPTION "PNG file checker"
        HOMEPAGE_URL "http://www.libpng.org/pub/png/apps/pngcheck.html"
        LANGUAGES C
)

# Options for zlib support
option(PNGCHECK_USE_ZLIB "Enable zlib support" ON)
option(PNGCHECK_USE_SYSTEM_ZLIB "Use the system-installed zlib" ON)

# Source files
set(PNGCHECK_SOURCES pngcheck.c)

# Executables
add_executable(pngcheck ${PNGCHECK_SOURCES})

# Dependency handling
if(PNGCHECK_USE_ZLIB)
    if(PNGCHECK_USE_SYSTEM_ZLIB)
        find_package(ZLIB)
        if(ZLIB_FOUND)
            target_link_libraries(pngcheck PRIVATE ZLIB::ZLIB)
        else()
            message(WARNING "System zlib not found, falling back to FetchContent")
            set(PNGCHECK_USE_SYSTEM_ZLIB OFF)
        endif()
    endif()
    if(NOT PNGCHECK_USE_SYSTEM_ZLIB)
        include(FetchContent)
        FetchContent_Declare(zlib
                             GIT_REPOSITORY https://github.com/madler/zlib.git
                             GIT_TAG v1.3.1
        )
        FetchContent_MakeAvailable(zlib)
        target_link_libraries(pngcheck PRIVATE zlibstatic)
    endif()
    target_compile_definitions(pngcheck PRIVATE USE_ZLIB)
endif()

# Compiler-specific options
if(MSVC)
    # Auto-expand the wildcard arguments
    # http://msdn.microsoft.com/en-us/library/8bch7bkk.aspx
    set(CMAKE_EXE_LINKER_FLAGS "setargv.obj ${CMAKE_EXE_LINKER_FLAGS}")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    # Enable commonly-used warning options by default
    set(CMAKE_CXX_FLAGS "-Wall -Wextra -Wundef ${CMAKE_CXX_FLAGS}")
endif()

# Installation rules
include(GNUInstallDirs)
install(TARGETS pngcheck
        RUNTIME
        DESTINATION ${CMAKE_INSTALL_BINDIR}
        COMPONENT Runtime
)
install(FILES pngcheck.1
        DESTINATION ${CMAKE_INSTALL_MANDIR}/man1
        COMPONENT Documentation
)

# Package generation
set(CPACK_PACKAGE_NAME ${PROJECT_NAME})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})
include(CPack)
