cmake_minimum_required(VERSION 3.24)

project(
  Gumbo
  LANGUAGES C CXX
  VERSION 0.10.1)

set(LIBRARY_NAME gumbo)

if(PROJECT_IS_TOP_LEVEL)
  get_property(isMultiConfig GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
  if(isMultiConfig)
    foreach(config IN LISTS CMAKE_CONFIGURATION_TYPES)
      set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${config} "${CMAKE_CURRENT_BINARY_DIR}/lib")
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${config} "${CMAKE_CURRENT_BINARY_DIR}/lib")
      set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${config} "${CMAKE_CURRENT_BINARY_DIR}/bin")
    endforeach()
  else()
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/lib")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin")
  endif()

  set(CMAKE_C_EXTENSIONS OFF)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

add_library(${LIBRARY_NAME})
target_sources(
  ${LIBRARY_NAME}
  PRIVATE src/attribute.c
          src/char_ref.c
          src/error.c
          src/parser.c
          src/string_buffer.c
          src/string_piece.c
          src/tag.c
          src/tokenizer.c
          src/utf8.c
          src/util.c
          src/vector.c)
set(${PROJECT_NAME}PublicHeaders # cmake-format: sortable
                                 src/gumbo.h src/tag_enum.h)
target_include_directories(
  ${LIBRARY_NAME}
  PRIVATE src
  PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
         "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")
target_compile_options(${LIBRARY_NAME} PRIVATE -Wall)
target_compile_features(${LIBRARY_NAME} PUBLIC c_std_99)
target_link_options(${LIBRARY_NAME} PRIVATE "LINKER:-no-undefined")
set_target_properties(${LIBRARY_NAME} PROPERTIES VERSION "1.0.0" PUBLIC_HEADER
                                                                 "${${PROJECT_NAME}PublicHeaders}")

include(GNUInstallDirs)
install(
  TARGETS ${LIBRARY_NAME}
  EXPORT ${PROJECT_NAME}Targets
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT shared_lib
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT static_lib
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT executable
  PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${LIBRARY_NAME}" COMPONENT headers)

set(${PROJECT_NAME}CmakeFilesDest "${CMAKE_INSTALL_LIBDIR}/cmake/${LIBRARY_NAME}")

install(
  EXPORT ${PROJECT_NAME}Targets
  NAMESPACE ${PROJECT_NAME}::
  FILE ${PROJECT_NAME}Targets.cmake
  DESTINATION "${${PROJECT_NAME}CmakeFilesDest}")

file(
  WRITE "${CMAKE_CURRENT_BINARY_DIR}/Config.cmake.in"
  [[
@PACKAGE_INIT@

include( "${CMAKE_CURRENT_LIST_DIR}/GumboTargets.cmake" )
]])

include(CMakePackageConfigHelpers)

configure_package_config_file(
  "${CMAKE_CURRENT_BINARY_DIR}/Config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
  INSTALL_DESTINATION "${${PROJECT_NAME}CmakeFilesDest}"
  NO_SET_AND_CHECK_MACRO NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
  VERSION "${${PROJECT_NAME}_VERSION}"
  COMPATIBILITY AnyNewerVersion)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION "${${PROJECT_NAME}CmakeFilesDest}")

export(EXPORT ${PROJECT_NAME}Targets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")
