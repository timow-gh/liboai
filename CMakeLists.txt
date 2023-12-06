cmake_minimum_required(VERSION 3.24)

project(oai
        VERSION 1.0.0)

option(${PROJECT_NAME}_INSTALL "Install project targets" ON)

find_package(nlohmann_json CONFIG REQUIRED)
find_package(CURL REQUIRED)

add_library(oai_lib
        liboai/components/audio.cpp
        liboai/components/azure.cpp
        liboai/components/chat.cpp
        liboai/components/completions.cpp
        liboai/components/edits.cpp
        liboai/components/embeddings.cpp
        liboai/components/files.cpp
        liboai/components/fine_tunes.cpp
        liboai/components/images.cpp
        liboai/components/models.cpp
        liboai/components/moderations.cpp
        liboai/core/authorization.cpp
        liboai/core/netimpl.cpp
        liboai/core/response.cpp
        liboai/include/liboai.h
)
add_library(oai::oai ALIAS oai_lib)
target_compile_features(oai_lib PRIVATE cxx_std_17)
target_include_directories(oai_lib PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/liboai/include>)
target_link_libraries(oai_lib PRIVATE nlohmann_json::nlohmann_json CURL::libcurl)

if (${PROJECT_NAME}_INSTALL)
    set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT oai)

    # Install (i.e. copy) the library headers
    install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/liboai/include/ DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/liboai")

    # Install the library target
    # oaiTargetsExportSet is the name of the export set that is created by the call to install(EXPORT)
    install(TARGETS oai_lib
            EXPORT oai_libTargetsExportSet
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
            # CMake doc on INCLUDES DESTINATION:
            # This option specifies a list of directories which will be added to the INTERFACE_INCLUDE_DIRECTORIES
            # target property of the <targets> when exported by the install(EXPORT) command. If a relative path is
            # specified, it is treated as relative to the $<INSTALL_PREFIX>.
            INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

    include(${CMAKE_SOURCE_DIR}/cmake/setup_export_set_installation.cmake)
    setup_export_set_installation(
        ${PROJECT_NAME}
        oai_libTargetsExportSet
        Config.cmake.in
        ${PROJECT_VERSION_MAJOR} ${PROJECT_VERSION_MINOR} ${PROJECT_VERSION_PATCH}
    )
endif ()