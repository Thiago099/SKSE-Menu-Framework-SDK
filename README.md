## Instalation

Install both mods from the `Mods` folder with your favorite mod manager, but only enable the right one for the build type you are doing. The DEV version is only for debug build, using the wrong version will probably cause a ctd when the menu is open.

Put the contents of the part two on your skyrim root folder

## Using this mod's template

Clone [this](https://github.com/Thiago099/SKSEMenuFrameworkTemplate) repository and follow the instructions on the readme.md

## Configuring an existing project

Add the following code to your cmake lists, and either update the `addMenuFramework("$ENV{SKYRIM_MODS_FOLDER}" "$ENV{IMGUI_LIB_FOLDER}")` function call with your paths, or set up these two environment variables:

- IMGUI_LIB_FOLDER

  Clone this  repository, to somewhere safe and adds its ImGui folder path to this environment variable on Windows.

- SKYRIM_MODS_FOLDER

  The skyrim mods folder is required to find the Menu framework library

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
## Usage

Define the render function for your menu entry. Here is an example:

```cpp
void __stdcall UI::Example1::Render() {
    ImGui::InputScalar("form id", ImGuiDataType_U32, &AddFormId, NULL, NULL, "%08X");

    if (ImGui::Button("Search")) {
        LookupForm();
    }

    if (AddBoundObject) {
        ImGui::Text("How much %s would you like to add?", AddBoundObject->GetName());
        ImGui::SliderInt("number", &Configuration::Example1::Number, 1, 100);
        if (ImGui::Button("Add")) {
            auto player = RE::PlayerCharacter::GetSingleton()->As<RE::TESObjectREFR>();
            player->AddObjectToContainer(AddBoundObject, nullptr, Configuration::Example1::Number, nullptr);
        }
    } else {
        ImGui::Text("Form not found");
    }
}
```

Then, import the SKSEMenuFramework.h into your header file, i reccomend you creating a UI.cpp/UI.h 

```cpp
#include "SKSEMenuFramework.h"
```

You should create a function in order to register your menu entries:

Before registering any entries, you should choose a section for your menu to be in. It is recommended that you use your mod name as the section name to keep things organized

```cpp
SKSEMenuFramework::SetSection("<menu section name>");
```

Register your menu entry, it will be a page on the Mod Control Panel

```cpp
SKSEMenuFramework::AddSectionItem("Add Item", Example1::Render);
```
Here is what this example will look like:

![image](https://github.com/Thiago099/SKSE-Menu-Framework-SDK/assets/66787043/8ebcd191-55a3-498b-bf36-0ca7337eff3a)

## Creating your own windows

Define your window render function

```cpp
void __stdcall UI::Example2::RenderWindow() {
    auto viewport = ImGui::GetMainViewport();
    ImGui::SetNextWindowPos(viewport->GetCenter(), ImGuiCond_Appearing, ImVec2{0.5f, 0.5f});
    ImGui::SetNextWindowSize(ImVec2{viewport->Size.x * 0.4f, viewport->Size.y * 0.4f}, ImGuiCond_Appearing);
    ImGui::Begin("My First Tool##MenuEntiryFromMod",nullptr, ImGuiWindowFlags_MenuBar); // If two mods have the same window name, and they open at the same time.
                                                                                         // The window content will be merged, is good practice to add ##ModName after the window name.
    if (ImGui::BeginMenuBar()) {
        if (ImGui::BeginMenu("File")) {
            if (ImGui::MenuItem("Open..", "Ctrl+O")) { /* Do stuff */
            }
            if (ImGui::MenuItem("Save", "Ctrl+S")) { /* Do stuff */
            }
            if (ImGui::MenuItem("Close", "Ctrl+W")) {
                ExampleWindow->IsOpen = false;
            }
            ImGui::EndMenu();
        }
        ImGui::EndMenuBar();
    }
    if (ImGui::Button("Close Window")) {
        ExampleWindow->IsOpen = false;
    }
    ImGui::End();
}
```

Register your window. The register method will return an object with which you can set the `IsOpen` property from anywhere to open and close your window

```cpp
UI::Example2::ExampleWindow = SKSEMenuFramework::AddWindow(Example2::RenderWindow);
UI::Example2::ExampleWindow->IsOpen = true; 
```

Here is the example header

```cpp
namespace Example2{
void __stdcall RenderWindow();
inline MENU_WINDOW ExampleWindow;
}
```

Here is what your window will look like:

![image](https://github.com/Thiago099/SKSE-Menu-Framework-SDK/assets/66787043/c301cc1b-d435-47ad-9bdc-a635fa385986)
