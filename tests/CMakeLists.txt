cmake_minimum_required(VERSION 2.8.11)

# On Windows, we must pair Debug and Release appropriately
if (WIN32)
   # For Windows, since 32-bit and 64-bit items can co-exist, we build each in its own build directory.
   # 32-bit target data goes in build32, and 64-bit target data goes into build.  So, include/link the
   # appropriate data at build time.
   if (CMAKE_CL_64)
      set (BUILDTGT_DIR build)
   else ()
      set (BUILDTGT_DIR build32)
   endif()
else()
    include (FindPkgConfig)
    if (BUILD_WSI_XCB_SUPPORT)
        find_package(XCB REQUIRED)
    endif()
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake")

find_package(ImageMagick COMPONENTS MagickWand)
if(NOT ImageMagick_FOUND)
    if(NOT WIN32)
        message(FATAL_ERROR "Missing ImageMagick library: sudo apt-get install libmagickwand-dev")
    else()
        message(FATAL_ERROR "Missing ImageMagick library: Get from http://www.imagemagick.org/script/binary-releases.php.  Be sure to check box to 'Install development headers and libraries'")
    endif()
endif()

if(WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_CRT_SECURE_NO_WARNINGS -D_USE_MATH_DEFINES -DMAGICKCORE_HDRI_ENABLE=0 -DMAGICKCORE_QUANTUM_DEPTH=16")

    # If MSVC, disable some signed/unsigned mismatch warnings.
    if (MSVC)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4267")
    endif()

else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -DMAGICKCORE_HDRI_ENABLE=0 -DMAGICKCORE_QUANTUM_DEPTH=16")
endif()

set (LIBGLM_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/libs)

set(COMMON_CPP
    vkrenderframework.cpp
    vktestbinding.cpp
    vktestframework.cpp
    test_environment.cpp
   )

include_directories(
    "${PROJECT_SOURCE_DIR}/tests/gtest-1.7.0/include"
    "${PROJECT_SOURCE_DIR}/icd/common"
    "${PROJECT_SOURCE_DIR}/layers"
    ${XCB_INCLUDE_DIRS}
    ${GLSLANG_SPIRV_INCLUDE_DIR}
    ${LIBGLM_INCLUDE_DIR}
    ${ImageMagick_INCLUDE_DIRS}
    )

if (NOT WIN32)
    # extra setup for out-of-tree builds
    if (NOT (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_CURRENT_BINARY_DIR))
        add_custom_target(binary-dir-symlinks ALL
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/run_all_tests.sh
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/run_wrap_objects_tests.sh
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/run_loader_tests.sh
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/run_extra_loader_tests.sh
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/vkvalidatelayerdoc.sh
            # Files unique to VulkanTools go below this line
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/golden
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/run_all_tests_with_layers.sh
            COMMAND ln -sf ${CMAKE_CURRENT_SOURCE_DIR}/vktracereplay.sh
            VERBATIM
            )
    endif()
else()
    if (NOT (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_CURRENT_BINARY_DIR))
        FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/golden GOLDEN)
        FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/_run_all_tests.ps1 RUN_ALL)
        FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/_run_all_tests_with_layers.ps1 RUN_ALL_WITH_LAYERS)
        FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/_vktracereplay.ps1 VKTRACEREPLAY)
        FILE(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/_vkvalidatelayerdoc.ps1 VKVALIDATELAYERDOC)
        add_custom_target(binary-dir-symlinks ALL
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${RUN_ALL} _run_all_tests.ps1
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${RUN_ALL_WITH_LAYERS} _run_all_tests_with_layers.ps1
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VKTRACEREPLAY} _vktracereplay.ps1
            COMMAND ${CMAKE_COMMAND} -E copy_if_different ${VKVALIDATELAYERDOC} _vkvalidatelayerdoc.ps1
            COMMAND IF NOT EXIST golden mklink /J golden ${GOLDEN}
            VERBATIM
            )
    endif()
endif()

if(WIN32)
   set (LIBVK "vulkan-${MAJOR}")
   set (TEST_LIBRARIES ${GLSLANG_LIBRARIES} ${ImageMagick_LIBRARIES})
elseif(UNIX)
   set (LIBVK "vulkan")
   set (TEST_LIBRARIES ${GLSLANG_LIBRARIES} ${ImageMagick_LIBRARIES} ${XCB_LIBRARIES} ${X11_LIBRARIES})
else()
endif()

add_executable(vkbase init.cpp ${COMMON_CPP})
set_target_properties(vkbase
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vkbase ${LIBVK} gtest gtest_main ${TEST_LIBRARIES})

add_executable(vk_image_tests image_tests.cpp ${COMMON_CPP})
set_target_properties(vk_image_tests
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vk_image_tests ${LIBVK} gtest gtest_main ${TEST_LIBRARIES})

add_executable(vk_render_tests render_tests.cpp ${COMMON_CPP})
set_target_properties(vk_render_tests
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vk_render_tests ${LIBVK} gtest gtest_main ${TEST_LIBRARIES})

add_executable(vk_blit_tests blit_tests.cpp ${COMMON_CPP})
set_target_properties(vk_blit_tests
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vk_blit_tests ${LIBVK} gtest gtest_main ${TEST_LIBRARIES})

add_executable(vk_layer_validation_tests layer_validation_tests.cpp ${COMMON_CPP})
set_target_properties(vk_layer_validation_tests
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vk_layer_validation_tests ${LIBVK} gtest gtest_main VkLayer_utils ${TEST_LIBRARIES})

add_executable(vk_loader_validation_tests loader_validation_tests.cpp ${COMMON_CPP})
set_target_properties(vk_loader_validation_tests
   PROPERTIES
   COMPILE_DEFINITIONS "GTEST_LINKED_AS_SHARED_LIBRARY=1")
target_link_libraries(vk_loader_validation_tests ${LIBVK} gtest gtest_main VkLayer_utils ${TEST_LIBRARIES})

add_subdirectory(gtest-1.7.0)
add_subdirectory(layers)
