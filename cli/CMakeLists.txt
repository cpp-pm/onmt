hunter_add_package(Boost COMPONENTS program_options)
find_package(Boost CONFIG REQUIRED COMPONENTS program_options)

add_executable(translate
  translate.cc
  BatchReader.cc
  BatchWriter.cc
  )

target_link_libraries(translate
  ${PROJECT_NAME}
  Boost::program_options
  )

install(
    TARGETS translate
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)
