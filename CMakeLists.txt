cmake_minimum_required(VERSION 2.8)
set( CMAKE_MODULE_PATH  "${CMAKE_CURRENT_SOURCE_DIR}/cmake" )

get_filename_component(list_file_path ${CMAKE_CURRENT_LIST_FILE} PATH)
string(REGEX MATCH "[a-z._-]*$" module_id ${list_file_path})
message("configuring vsxu module            " ${module_id})

project (${module_id})

find_package(VSXu REQUIRED)
include_directories( ${VSXU_INCLUDE_DIRS} )
link_directories(${VSXu_LIBRARY_DIRS})

set(AUDIO_LIBRARIES "")
set(WITH_MIDI FALSE)

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  find_package(PULSEAUDIO)
  if(PULSEAUDIO_FOUND)
    add_definitions(-D__LINUX_PULSE__)

    include_directories(${PULSEAUDIO_INCLUDE_DIRS})
    set( AUDIO_LIBRARIES ${PULSEAUDIO_LIBRARY} ${PULSEAUDIO_SIMPLE_LIBRARY})
  endif(PULSEAUDIO_FOUND)
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

if(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  find_package(ALSA)
  if(ALSA_FOUND)
    add_definitions(-D__LINUX_ALSA__)
    add_definitions( -DWITH_MIDI )
    set( WITH_MIDI TRUE )

    include_directories(${ALSA_INCLUDE_DIRS})
    set( AUDIO_LIBRARIES ${AUDIO_LIBRARIES} ${ALSA_LIBRARY} )

  endif(ALSA_FOUND)
endif(${CMAKE_SYSTEM_NAME} MATCHES "Linux")


if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  find_library(COREAUDIO_LIBRARY CoreAudio)
  if(COREAUDIO_LIBRARY_FOUND)
    add_definitions(-D__MACOSX_CORE__)
    add_definitions(-D__OS_MACOSX__)

    set( AUDIO_LIBRARIES ${AUDIO_LIBRARIES} ${COREAUDIO_LIBRARY})
  endif(COREAUDIO_LIBRARY_FOUND)
endif(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")


if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  add_definitions(-D__WINDOWS_DS__)
  set( AUDIO_LIBRARIES ${AUDIO_LIBRARIES} winmm dsound)
endif(${CMAKE_SYSTEM_NAME} MATCHES "Windows")

#TODO: Add in the needed definitions for each Audio/Midi Library on each platform
message("Audio libraries: " ${AUDIO_LIBRARIES})
if(AUDIO_LIBRARIES)
  find_package(PTHREAD)
  #include (../cmake_globals.txt)
  include_directories ( lib/
                        lib/fftreal/ )
  set(SOURCES main.cpp
              input/input_audio_raw.cpp
              input/input_audio_mediaplayer.cpp
              lib/fftreal/fftreal.cpp
              lib/RtAudio/RtAudio.cpp
              lib/RtMidi/RtMidi.cpp )

  if(WITH_MIDI)
    list(APPEND SOURCES input/input_audio_midi.cpp)
  endif(WITH_MIDI)

  add_library(${module_id} MODULE ${SOURCES})
  #include(../cmake_suffix.txt)
  target_link_libraries(${module_id} ${AUDIO_LIBRARIES} ${PTHREAD_LIBRARY} ${VSXU_LIBRARIES})


  #Now Install the magic  files
  if(WIN32)
    target_link_libraries(${module_id} gdi32)
    install(TARGETS ${module_id} DESTINATION ${VSXU_LIBRARY_DIRS}/plugins COMPONENT engine)
  endif(WIN32)

  if (UNIX)
      install(TARGETS ${module_id} DESTINATION ${VSXU_LIBRARY_DIRS}/vsxu/plugins)
  endif(UNIX)

  # Clean up the lib prefix from the module that would be installed
  set_target_properties(
    ${module_id}
      PROPERTIES
        PREFIX ""
  )

  #Adding a nice shiny uninstall target
  configure_file(
      "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
      "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
      IMMEDIATE @ONLY)

  add_custom_target(uninstall
      COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

endif(AUDIO_LIBRARIES)