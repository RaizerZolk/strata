###
# Copyright (c) 2009, Paul Gideon Dann
#
# Permission to use, copy, modify, and/or distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
###

# Ensure that 7z is in the path
find_path(7Z_PATH 7z.exe)
if (NOT 7Z_PATH)
    message(WARNING "Unable to find 7z in the PATH. The zip archive will be disabled.")
endif()

# Paths to Qt DLLs
get_target_property(QtCore_LOCATION Qt5::Core LOCATION)
get_filename_component(QT_DLL_DIR ${QtCore_LOCATION} PATH)

# Compiler Runtime DLLs
if (MSVC)
    # Visual Studio
    set(CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_SKIP true)
    include(InstallRequiredSystemLibraries)
    install(
        FILES ${CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS} 
        DESTINATION .
        COMPONENT dependencies
        )
else()
    # MinGW
    get_filename_component(MINGW_DLL_DIR ${CMAKE_CXX_COMPILER} PATH)
    install(FILES
        ${MINGW_DLL_DIR}/libgcc_s_dw2-1.dll
        ${MINGW_DLL_DIR}/libstdc++-6.dll
        ${QT_DLL_DIR}/libwinpthread-1.dll
        DESTINATION .
        COMPONENT dependencies
        )
endif()

# Qt Libraries
install(FILES
    ${QT_DLL_DIR}/Qt5Core.dll
    ${QT_DLL_DIR}/Qt5Gui.dll
    ${QT_DLL_DIR}/Qt5OpenGL.dll
    ${QT_DLL_DIR}/Qt5PrintSupport.dll
    ${QT_DLL_DIR}/Qt5Svg.dll
    ${QT_DLL_DIR}/Qt5Widgets.dll
    ${QT_DLL_DIR}/Qt5Xml.dll
    ${CMAKE_SOURCE_DIR}/win/qt.conf
    DESTINATION .
    COMPONENT dependencies
    )
# Qwt Library
get_filename_component(QWT_DLL_DIR ${QWT_LIBRARY} PATH)
install(FILES
    ${QWT_DLL_DIR}/qwt.dll
    DESTINATION .
    COMPONENT dependencies
    )
# GSL Library
get_filename_component(GSL_DLL_DIR ${GSL_LIBRARY} PATH)
install(FILES
    ${GSL_DLL_DIR}/../bin/gsl.dll
    ${GSL_DLL_DIR}/../bin/gslcblas.dll
    DESTINATION .
    COMPONENT dependencies
    )  
# Qt Platform Plugin
install(FILES
    ${QT_DLL_DIR}/../plugins/platforms/qwindows.dll
    DESTINATION plugins/platforms
    COMPONENT dependencies
    )
install(FILES
    ${QT_DLL_DIR}/../plugins/imageformats/qsvg.dll
    DESTINATION plugins/imageformats
    COMPONENT dependencies
    )
install(FILES
    ${QT_DLL_DIR}/../plugins/iconengines/qsvgicon.dll
    DESTINATION plugins/iconengines
    COMPONENT dependencies
    )

if (7Z_PATH)
    message(${CMAKE_BINARY_DIR})
    message(${CMAKE_SOURCE_DIR})
    message(${CMAKE_CURRENT_BINARY_DIR})
    message(${CMAKE_CURRENT_SOURCE_DIR})
    add_custom_target(archive
        COMMAND 7z a ${CMAKE_BINARY_DIR}/Strata-$ENV{platform}-v${PROJECT_VERSION}-${PROJECT_GITHASH}.zip *
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/dist
        VERBATIM
        )
endif()

# Qt Installer Framework 
set(CPACK_PACKAGE_NAME Strata)
set(CPACK_PACKAGE_FILE_NAME 
    Strata-$ENV{platform}-v${PROJECT_VERSION}-${PROJECT_GITHASH}
)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Installation Tool")
set(CPACK_PACKAGE_VERSION "1.0.0") # Version of installer
set(CPACK_COMPONENTS_ALL executable dependencies manual example)
set(CPACK_IFW_PACKAGE_START_MENU_DIRECTORY Strata)
set(CPACK_GENERATOR IFW)
set(CPACK_IFW_VERBOSE ON)

include(CPack REQUIRED)
include(CPackIFW REQUIRED)

cpack_add_component(
    executable
    DISPLAY_NAME "Executable"
    DESCRIPTION "Strata executable"
    REQUIRED
)

cpack_ifw_configure_component(
    executable
    FORCED_INSTALLATION
    NAME arkottke.strata.executable
    VERSION ${PROJECT_VERSION} # Version of component
    LICENSES License ${CMAKE_SOURCE_DIR}/LICENSE
    DEFAULT TRUE
)

cpack_add_component(
    dependencies
    DISPLAY_NAME "Dependencies"
    DESCRIPTION "Strata dependencies"
    REQUIRED
)

cpack_add_component(
    manual
    NAME arkottke.strata.manual
    DISPLAY_NAME "User's manual"
    DESCRIPTION "User's manual"
)

cpack_add_component(
    example
    NAME arkottke.strata.example
    DISPLAY_NAME "Example"
    DESCRIPTION "Example target and motions"
)
