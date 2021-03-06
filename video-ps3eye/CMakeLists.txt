add_compile_definitions(PS3EYE_DEBUG)

if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/ps3eye-driver/CMakeLists.txt")
    add_subdirectory("ps3eye-driver")

    if(NOT MSVC)
        if(PKG_CONFIG_FOUND)
            pkg_check_modules(libusb "libusb-1.0" QUIET)
        endif()
        if(libusb_FOUND)
            include_directories(SYSTEM ${libusb_INCLUDE_DIRS})
            link_libraries(${libusb_LIBRARIES})
            link_directories(${libusb_LIBRARY_DIRS})
        endif()
    else()
        set(SDK_LIBUSB CACHE PATH "")
        if(SDK_LIBUSB)
            set(libusb_FOUND TRUE)
            link_directories("${SDK_LIBUSB}")
            include_directories(SYSTEM "${SDK_LIBUSB}")
        endif()
    endif()
endif()

if(TARGET ps3eye-driver)
    otr_module(video-ps3eye)
    link_libraries(ps3eye-driver)
    add_executable(ps3eye-subprocess "wrapper.cxx")

    # For a odd reason on Travis the output directory is an absolute path to root
    #set_target_properties(ps3eye-subprocess
     #   PROPERTIES
     #   RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/build"
    #)

    if(WIN32)
        set(path "${SDK_LIBUSB}/libusb-1.0.dll")
        if(EXISTS "${path}")
            otr_install_lib("${path}" "${opentrack-hier-pfx}")
        endif()
    endif()
endif()

if(TARGET ps3eye-sdl)
    install(TARGETS "ps3eye-sdl" DESTINATION "${opentrack-hier-pfx}")
    if(WIN32)
        foreach(k ${SDL2_LIBRARIES})
            get_filename_component(path "${k}" PATH)
            set(lib "${path}/SDL2.dll")
            if(EXISTS "${lib}")
                otr_install_lib("${lib}" "${opentrack-hier-pfx}")
                break()
            endif()
        endforeach()
    endif()
endif()

if(TARGET ps3eye-mode-test)
    install(TARGETS "ps3eye-mode-test" DESTINATION "${opentrack-hier-pfx}")
endif()

if(TARGET ps3eye-frame-test)
    install(TARGETS "ps3eye-frame-test" DESTINATION "${opentrack-hier-pfx}")
endif()
