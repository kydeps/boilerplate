cmake_minimum_required(VERSION 3.22)
project(kydeps_boilerplate)

option(KYDEPS_BOILERPLATE_GITIGNORE "enable .gitignore symlink" ON)
option(KYDEPS_BOILERPLATE_CLANG_FORMAT "enable .clang-format symlink" ON)
option(KYDEPS_BOILERPLATE_CLANG_TIDY "enable .clang-format symlink" ON)
option(KYDEPS_BOILERPLATE_CMAKE_LINT "enable .cmake-lint symlink" ON)
option(KYDEPS_BOILERPLATE_CMAKE_INSTALL "enable cmake --install ..." ON)
option(KYDEPS_BOILERPLATE_CIRCLECI "enable .circleci configuration" ON)
option(KYDEPS_BOILERPLATE_LICENSE "enable adding LICENSE" ON)
option(KYDEPS_BOILERPLATE_README "enable adding README.md" ON)
option(KYDEPS_BOILERPLATE_SRC "enable the use of src directory" ON)

# if KYDEPS_TARGETS is defined, add the definitions... ... and use "main" unless
# the value is overriden outside

if("-${KYDEPS_DEFINITIONS_GIT_REPOSITORY}-" STREQUAL "--")
    set(KYDEPS_DEFINITIONS_GIT_REPOSITORY
        "https://github.com/kydeps/definitions.git"
        CACHE STRING "KyDeps definitions repository URL")
endif()

if("-${KYDEPS_DEFINITIONS_GIT_TAG}-" STREQUAL "--")
    set(KYDEPS_DEFINITIONS_GIT_TAG
        "main"
        CACHE STRING "KyDeps definitions repository revision")
endif()

function(CreateSymLink2 OPTION SOURCE TARGET)
    if(${OPTION})
        set(T "${CMAKE_SOURCE_DIR}/${TARGET}")
        message(STATUS "creating ${T}")
        if(EXISTS "${T}")
            file(REMOVE "${T}")
        endif()
        file(CREATE_LINK "${kydeps_boilerplate_SOURCE_DIR}/${SOURCE}" "${T}"
             SYMBOLIC)
    endif()
endfunction()

function(CreateSymLink OPTION TARGET)
    CreateSymLink2("${OPTION}" "${TARGET}" "${TARGET}")
endfunction()

function(KyDepsPackage)
    get_property(
        KYDEPS_PACKAGE_TARGET_EXISTS GLOBAL
        PROPERTY KYDEPS_PACKAGE_TARGET_EXISTS
        SET)

    if(KYDEPS_PACKAGE_TARGET_EXISTS)
        set(NS "${CMAKE_PROJECT_NAME}")

        install(
            EXPORT "${NS}-targets"
            FILE "${NS}-targets.cmake"
            NAMESPACE "${NS}::"
            DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${NS}")

        configure_package_config_file(
            "${CMAKE_SOURCE_DIR}/cmake/config.cmake.in"
            "${CMAKE_BINARY_DIR}/cmake/${NS}-config.cmake"
            INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${NS}")

        write_basic_package_version_file(
            "${CMAKE_BINARY_DIR}/cmake/${NS}-config-version.cmake"
            VERSION "${${NS}_VERSION}"
            COMPATIBILITY AnyNewerVersion)

        install(FILES "${CMAKE_BINARY_DIR}/cmake/${NS}-config.cmake"
                      "${CMAKE_BINARY_DIR}/cmake/${NS}-config-version.cmake"
                DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${NS}")
    else()
        message(
            STATUS
                "WARNING:

    No KyDepsPackage[Library]Target(...) seen
        -> Skipping install generation!
    Consider:
        * specifying install targets with KyDepsPackage[Library]Target(...), or
        * disabling install with set(KYDEPS_BOILERPLATE_CMAKE_INSTALL OFF)
            ")
    endif()
endfunction()

function(KyDepsPackageTarget TARGET)
    message(STATUS "KyDepsPackageTarget : ${TARGET}")
    set_property(GLOBAL PROPERTY KYDEPS_PACKAGE_TARGET_EXISTS ON)

    set_target_properties(
        "${TARGET}"
        PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib"
                   LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib"
                   RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin")

    install(
        TARGETS "${TARGET}"
        EXPORT "${CMAKE_PROJECT_NAME}-targets"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endfunction()

function(KyDepsPackageLibraryTarget TARGET INCLUDE_RELATIVE_PATH)
    add_library(${CMAKE_PROJECT_NAME}::${TARGET} ALIAS ${TARGET})

    set(INTERFACE_DIR
        "${CMAKE_CURRENT_BINARY_DIR}/include/${INCLUDE_RELATIVE_PATH}")

    file(MAKE_DIRECTORY "${INTERFACE_DIR}")
    file(REMOVE_RECURSE "${INTERFACE_DIR}")
    file(CREATE_LINK "${CMAKE_CURRENT_LIST_DIR}/include" "${INTERFACE_DIR}"
         SYMBOLIC)

    file(
        GLOB_RECURSE INCLUDES FOLLOW_SYMLINKS
        RELATIVE "${CMAKE_CURRENT_BINARY_DIR}/include"
        "${CMAKE_CURRENT_BINARY_DIR}/include/*")
    message(STATUS "${INCLUDES}")

    foreach(INCLUDE ${INCLUDES})
        cmake_path(GET INCLUDE PARENT_PATH INCLUDE_SUBDIR)
        install(FILES "${CMAKE_CURRENT_BINARY_DIR}/include/${INCLUDE}"
                DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${INCLUDE_SUBDIR}")
    endforeach()

    # install(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/include"
    #         DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/..")

    target_include_directories(
        "${TARGET}"
        PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>"
        PUBLIC "$<INSTALL_INTERFACE:include>")

    KyDepsPackageTarget("${TARGET}")
endfunction()

macro(KyDepsAddBoilerplate)
    CreateSymLink2(KYDEPS_BOILERPLATE_GITIGNORE ".gitignore.copy" ".gitignore")
    CreateSymLink(KYDEPS_BOILERPLATE_CLANG_FORMAT ".clang-format")
    CreateSymLink(KYDEPS_BOILERPLATE_CLANG_TIDY ".clang-tidy")
    CreateSymLink(KYDEPS_BOILERPLATE_CMAKE_LINT ".cmake-lint")

    if(KYDEPS_BOILERPLATE_CMAKE_LINT)
        file(MAKE_DIRECTORY "${CMAKE_SOURCE_DIR}/.vscode")
    endif()

    CreateSymLink(KYDEPS_BOILERPLATE_CMAKE_LINT ".vscode/tasks.json")

    if(KYDEPS_BOILERPLATE_CLANG_TIDY)
        find_program("CLANG_TIDY" "clang-tidy")
        if(EXISTS "${CLANG_TIDY}")
            message(STATUS "found clang-tidy @ ${CLANG_TIDY}")
            set(CMAKE_CXX_CLANG_TIDY
                "${CLANG_TIDY}" "-header-filter=.*"
                "-export-fixes=${CMAKE_BINARY_DIR}/clang-tidy.fixes")
        else()
            message(WARNING "clang-tidy not found :(")
        endif()
    endif()

    if(KYDEPS_BOILERPLATE_CMAKE_INSTALL)
        file(MAKE_DIRECTORY "${CMAKE_SOURCE_DIR}/cmake")
        if(NOT EXISTS "${CMAKE_SOURCE_DIR}/cmake/config.cmake.in")
            file(COPY_FILE
                 "${kydeps_boilerplate_SOURCE_DIR}/cmake/config.cmake.in"
                 "${CMAKE_SOURCE_DIR}/cmake/config.cmake.in")
        endif()

        include(GNUInstallDirs)
        include(CMakePackageConfigHelpers)

        cmake_language(DEFER DIRECTORY ${CMAKE_SOURCE_DIR} CALL KyDepsPackage)
    endif()

    if(KYDEPS_BOILERPLATE_CIRCLECI)
        file(MAKE_DIRECTORY "${CMAKE_SOURCE_DIR}/.circleci")
        if(NOT EXISTS "${CMAKE_SOURCE_DIR}/.circleci/config.yml")
            file(COPY_FILE
                 "${kydeps_boilerplate_SOURCE_DIR}/.circleci/config.yml"
                 "${CMAKE_SOURCE_DIR}/.circleci/config.yml")
        endif()
    endif()

    if(NOT "-${KYDEPS_TARGETS}-" STREQUAL "--")
        include(FetchContent)
        FetchContent_Declare(
            kydeps_definitions
            #
            # NOTE: the repo and tag below may be something you want to override
            #
            GIT_REPOSITORY "${KYDEPS_DEFINITIONS_GIT_REPOSITORY}"
            GIT_TAG "${KYDEPS_DEFINITIONS_GIT_TAG}")
        FetchContent_MakeAvailable(kydeps_definitions)
    endif()

    include(CTest)
    include(GoogleTest)
    enable_testing()

    if(KYDEPS_BOILERPLATE_LICENSE AND NOT EXISTS "${CMAKE_SOURCE_DIR}/LICENSE")
        file(COPY_FILE "${kydeps_boilerplate_SOURCE_DIR}/LICENSE.copy"
             "${CMAKE_SOURCE_DIR}/LICENSE")
    endif()

    if(KYDEPS_BOILERPLATE_README AND NOT EXISTS "${CMAKE_SOURCE_DIR}/README.md")
        file(COPY_FILE "${kydeps_boilerplate_SOURCE_DIR}/README.md.copy"
             "${CMAKE_SOURCE_DIR}/README.md")
    endif()

    if(KYDEPS_BOILERPLATE_SRC AND NOT EXISTS "${CMAKE_SOURCE_DIR}/src")
        message(STATUS "creating sample code in ${CMAKE_SOURCE_DIR}/src...")
        file(COPY "${kydeps_boilerplate_SOURCE_DIR}/src"
             DESTINATION "${CMAKE_SOURCE_DIR}")
    endif()

endmacro()
