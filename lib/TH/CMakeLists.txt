include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  )

set(PUBLIC_HEADERS
  THGeneral.h
  THFile.h
  THDiskFile.h
  )

add_library(TH
  THGeneral.c
  THFile.c
  THDiskFile.c
  )

target_link_libraries(TH PUBLIC Eigen3::Eigen)

install(
    TARGETS TH
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
  FILES ${PUBLIC_HEADERS}
  DESTINATION include/TH
  )
