## Instalation

Install both mods with your favorite mod manager, but only enable the right one from the build you are doing. The DEV version is only for debug build, using the wrong version will probably cause a ctd when the menu is open.

Put the contents of the part two on your skyrim root folder

## Using this mod's template

Clone [this](https://github.com/Thiago099/SKSEMenuFrameworkTemplate) repository and follow the instructions on the readme.md

## Configuring existing project

Add the following code to your cmake lists, and either update the `addMenuFramework("$ENV{SKYRIM_MODS_FOLDER}" "$ENV{IMGUI_LIB_FOLDER}")` function call with your paths, or set up these two environment variables:

SKYRIM_MODS_FOLDER

IMGUI_LIB_FOLDER

[How to set up envioriment variables](https://gist.github.com/Thiago099/b45ec7832fb754325b29a61006bcd10c)


```cmake
function(addMenuFramework MENU_FRAMEWORK_FOLDER IMGUI_FOLDER)

    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(IMGUI_INSTALL_DIR "${IMGUI_FOLDER}/debug")
    else()
        set(IMGUI_INSTALL_DIR "${IMGUI_FOLDER}/release")
    endif()

    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        set(PROJECT_SUFFIX " DEV")
    else()
        set(PROJECT_SUFFIX "")
    endif()

    set(ImGui_DIR "${IMGUI_INSTALL_DIR}/lib/cmake")
    list(APPEND CMAKE_PREFIX_PATH ${ImGui_DIR})
    find_package(ImGui CONFIG REQUIRED)
    target_link_libraries(${PROJECT_NAME} PRIVATE ImGui::imgui)

    add_library(SKSEMenuFramework SHARED IMPORTED GLOBAL)
    set_target_properties(SKSEMenuFramework PROPERTIES
        IMPORTED_LOCATION "${MENU_FRAMEWORK_FOLDER}/SKSE Menu Framework${PROJECT_SUFFIX}/SKSE/Plugins/SKSEMenuFramework.dll"
        IMPORTED_IMPLIB "${MENU_FRAMEWORK_FOLDER}/SKSE Menu Framework${PROJECT_SUFFIX}/SKSE/Plugins/SKSEMenuFramework.lib"
    )

    target_include_directories(
	    ${PROJECT_NAME}
	    PRIVATE
        "${MENU_FRAMEWORK_FOLDER}/SKSE Menu Framework DEV/SKSE/Plugins"
    )

    target_link_libraries(${PROJECT_NAME} PRIVATE SKSEMenuFramework)

endfunction()

addMenuFramework("$ENV{SKYRIM_MODS_FOLDER}" "$ENV{IMGUI_LIB_FOLDER}")

```
