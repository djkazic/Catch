#[[

Catch, Formlabs fork
====================

Usage
-----

*   Include the directory containing this CMakeLists.txt into your project.
*   Link your test binaries against the build target, `Catch`.
*   The header `catch.hpp` should be visible in the sources of those bianries.
*   Use Catch as normal.

More details
------------

This CMakeLists.txt will check whether there is already a target called `Catch`
defined. If so, it will try to determine whether that version of Catch is
compatible with this one. The way this is done is that the `_CATCH_GIT_REVISION`
property of the target is queried, and compared for equality with the Git
revision of this repository. If that target property is not found, or does not
match this repository's Git revision, then a warning message is printed.

If no `Catch` target exists, one will be defined as normal.

This is all done to support, for example, the scenario that your repository uses
submodules A and B, and both A and B use Catch. If both A and B include Catch
using this repository, and their Catch revisions are in sync, there will be no
problem. If the revisions are out-of-sync, then you will get a warning message,
and if the Catch versions are actually incompatible then your test code may
break.

]]

cmake_minimum_required (VERSION 3.4)

find_package (Git REQUIRED)

execute_process (
    COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
    OUTPUT_VARIABLE _CATCH_GIT_REVISION
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

string (STRIP "${_CATCH_GIT_REVISION}" _CATCH_GIT_REVISION)

if (TARGET Catch)
    get_target_property (_OTHER_CATCH_INCLUDE_DIRECTORIES
        Catch INTERFACE_INCLUDE_DIRECTORIES
    )

    list (GET _OTHER_CATCH_INCLUDE_DIRECTORIES 0 _OTHER_CATCH_INCLUDE_DIRECTORY)

    execute_process (
        COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
        OUTPUT_VARIABLE _OTHER_CATCH_GIT_REVISION
        WORKING_DIRECTORY ${_OTHER_CATCH_INCLUDE_DIRECTORY}
    )

    string (STRIP "${_OTHER_CATCH_GIT_REVISION}" _OTHER_CATCH_GIT_REVISION)

    if (NOT ${_CATCH_GIT_REVISION} STREQUAL ${_OTHER_CATCH_GIT_REVISION})
        message (WARNING "
            Potentially incompatible versions of Catch found!
            The revision at ${CMAKE_CURRENT_SOURCE_DIR} is ${_CATCH_GIT_REVISION}.
            The revision of the existing Catch target is ${_OTHER_CATCH_GIT_REVISION}.
            The include directories of the existing Catch target are ${_OTHER_CATCH_INCLUDE_DIRECTORIES}.
            The other Catch target is the one that will be used.
        ")
    endif ()
else ()
    add_library (Catch INTERFACE)
    target_include_directories (Catch INTERFACE single_include)
endif ()
