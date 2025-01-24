cmake_minimum_required(VERSION 3.1)

###############################################################################
## Prepare VCPKG ##############################################################
###############################################################################
# Fetch VCPKG and make it available
message(STATUS "Fetching VCPKG...")
include(FetchContent)
FetchContent_Declare(
        vcpkg
        GIT_TAG 2022.11.14
        GIT_REPOSITORY https://github.com/microsoft/vcpkg.git)
FetchContent_MakeAvailable(vcpkg)

# we only need releas build
set(VCPKG_BUILD_TYPE release)

# Add VCPKG to CMake
set(CMAKE_TOOLCHAIN_FILE ${vcpkg_SOURCE_DIR}/scripts/buildsystems/vcpkg.cmake
        CACHE STRING "CMake toolchain file")

# setup include directories
include_directories("${CMAKE_BINARY_DIR}/vcpkg_installed/${VCPKG_TARGET_TRIPLET}/include/")


###############################################################################
## Project Config #############################################################
###############################################################################
project(RecastNavigation)

# lib versions
SET(SOVERSION 1)
set(LIB_VERSION 1.6.0)
string(REPLACE "." "," LIB_VERSION_NUM "${LIB_VERSION}.0")

set_property(GLOBAL PROPERTY CXX_STANDARD 98)

option(RECASTNAVIGATION_DEMO "Build demo" ON)
option(RECASTNAVIGATION_TESTS "Build tests" ON)
option(RECASTNAVIGATION_EXAMPLES "Build examples" ON)
option(RECASTNAVIGATION_DT_POLYREF64 "Use 64bit polyrefs instead of 32bit for Detour" OFF)
option(RECASTNAVIGATION_DT_VIRTUAL_QUERYFILTER "Use dynamic dispatch for dtQueryFilter in Detour to allow for custom filters" OFF)

if(MSVC AND BUILD_SHARED_LIBS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

configure_file(
    "${RecastNavigation_SOURCE_DIR}/version.h.in"
    "${RecastNavigation_BINARY_DIR}/version.h")
install(FILES "${RecastNavigation_BINARY_DIR}/version.h" DESTINATION
        ${CMAKE_INSTALL_INCLUDEDIR}/recastnavigation)

# Needed for recastnavigation.pc.in
set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix "\${prefix}")
set(libdir "\${exec_prefix}/${CMAKE_INSTALL_LIBDIR}")
set(bindir "\${exec_prefix}/${CMAKE_INSTALL_BINDIR}")
set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
set(PACKAGE_VERSION "${LIB_VERSION}")
if(RECASTNAVIGATION_DT_POLYREF64)
    set(PKG_CONFIG_CFLAGS "${PKG_CONFIG_CFLAGS} -DDT_POLYREF64")
endif()
if(RECASTNAVIGATION_DT_VIRTUAL_QUERYFILTER)
    set(PKG_CONFIG_CFLAGS "${PKG_CONFIG_CFLAGS} -DDT_VIRTUAL_QUERYFILTER")
endif()
configure_file(
        "${RecastNavigation_SOURCE_DIR}/recastnavigation.pc.in"
        "${RecastNavigation_BINARY_DIR}/recastnavigation.pc"
        @ONLY)
install(FILES "${RecastNavigation_BINARY_DIR}/recastnavigation.pc"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

add_subdirectory(DebugUtils)
add_subdirectory(Detour)
add_subdirectory(DetourCrowd)
add_subdirectory(DetourTileCache)
add_subdirectory(Recast)

configure_package_config_file(
    ${PROJECT_SOURCE_DIR}/recastnavigation-config.cmake.in
    recastnavigation-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/recastnavigation
)

write_basic_package_version_file(
    recastnavigation-config-version.cmake
    VERSION ${LIB_VERSION}
    COMPATIBILITY AnyNewerVersion
)

export(
    EXPORT recastnavigation-targets
    NAMESPACE RecastNavigation::
    FILE recastnavigation-targets.cmake
)

install(
    EXPORT recastnavigation-targets
    NAMESPACE RecastNavigation::
    FILE recastnavigation-targets.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/recastnavigation
)

install(
    FILES
        ${PROJECT_BINARY_DIR}/recastnavigation-config.cmake
        ${PROJECT_BINARY_DIR}/recastnavigation-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/recastnavigation
)

if (RECASTNAVIGATION_DEMO)
    add_subdirectory(RecastDemo)
endif ()

if (RECASTNAVIGATION_TESTS)
    enable_testing()
    add_subdirectory(Tests)
endif ()
