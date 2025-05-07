cmake_minimum_required(VERSION 3.25)
project(HCEngine)

#add_subdirectory(thirdparty/HCWindow)

include(FetchContent)
FetchContent_Declare(
        HC-Window
        GIT_REPOSITORY https://github.com/mora-hugo/HC-Window
        GIT_TAG v0.0.1
)
FetchContent_MakeAvailable(HC-Window)

# Initialize the list of required DLLs
set(ALL_REQUIRED_DLLS "")

# Set the C++ standard
set(CMAKE_CXX_STANDARD 20)

# Add the executable target
add_executable(HCEngine main.cpp)

# Collect DLLs
if(DEFINED HC_WINDOW_DLLS)
    list(APPEND ALL_REQUIRED_DLLS ${HC_WINDOW_DLLS})
endif()

# Link the executable to the HC-Window library
target_link_libraries(HCEngine PRIVATE HC-Window)

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    # Set up the deployment directory
    set(DEPLOY_DIR "${CMAKE_BINARY_DIR}/deploy/HCEngine")

    file(REMOVE_RECURSE ${DEPLOY_DIR})
    file(MAKE_DIRECTORY ${DEPLOY_DIR})

    # In Release mode, copy DLLs and executable to the deployment directory
    if(WIN32 AND ALL_REQUIRED_DLLS)
        list(REMOVE_DUPLICATES ALL_REQUIRED_DLLS)

        # Copy each DLL to the deployment directory
        foreach(DLL_PATH ${ALL_REQUIRED_DLLS})
            add_custom_command(TARGET HCEngine POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E copy_if_different
                    ${DLL_PATH}
                    ${DEPLOY_DIR}
                    COMMENT "Copying ${DLL_PATH} to ${DEPLOY_DIR}"
            )
        endforeach()
    endif()

    # Copy the executable to the deployment directory
    add_custom_command(TARGET HCEngine POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:HCEngine>
            ${DEPLOY_DIR}
            COMMENT "Copying HCEngine executable to ${DEPLOY_DIR}"
    )
else ()
    # In Debug mode, copy DLLs to the executable directory
    if(WIN32 AND ALL_REQUIRED_DLLS)
        list(REMOVE_DUPLICATES ALL_REQUIRED_DLLS)

        # Copy each DLL to the executable directory
        foreach(DLL_PATH ${ALL_REQUIRED_DLLS})
            add_custom_command(TARGET HCEngine POST_BUILD
                    COMMAND ${CMAKE_COMMAND} -E copy_if_different
                    ${DLL_PATH}
                    $<TARGET_FILE_DIR:HCEngine>
                    COMMENT "Copying ${DLL_PATH}"
            )
        endforeach()
    endif()
endif ()
