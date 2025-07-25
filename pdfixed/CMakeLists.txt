# Identify Thrift files
file(GLOB THRIFT_FILES "${CMAKE_CURRENT_SOURCE_DIR}/thrift/*.thrift")

# Set up build dirs
set(THRIFT_GEN_CPP_DIR ${CMAKE_CURRENT_BINARY_DIR}/gen-cpp)
set(THRIFT_GEN_PY_DIR ${CMAKE_CURRENT_BINARY_DIR}/gen-py)

# Generate Thrift sources
foreach(THRIFT_FILE ${THRIFT_FILES})
  generate_thrift(${THRIFT_FILE} THRIFT_CPP_FILES THRIFT_PY_NAMESPACE THRIFT_PY_FILES)
  list(APPEND ALL_THRIFT_CPP_FILES ${THRIFT_CPP_FILES})

  install(FILES
    ${THRIFT_PY_FILES}
    DESTINATION "${PY_SITE_PKG_DIR}/bm/pdfixed/${THRIFT_PY_NAMESPACE}"
  )
endforeach()

add_custom_target(genthrift_pdfixed DEPENDS ${ALL_THRIFT_CPP_FILES})

# Create libraries
add_library(bmpdfixed STATIC
  src/pd_notifications.cpp
  src/pd_helpers.cpp
  src/pd_pre.cpp
  src/pd_mirroring.cpp
  src/pd_static.cpp
  src/pd_swap.cpp
)
target_link_libraries(bmpdfixed
  runtimestubs
  simpleswitch_thrift
)
target_include_directories(bmpdfixed PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_BINARY_DIR}/thrift_src/gen-cpp
  ${CMAKE_BINARY_DIR}/targets/simple_switch/gen-cpp
)
add_dependencies(bmpdfixed genthrift_simpleswitch)

add_library(bmpdfixed_thrift STATIC
  thrift-src/pdfixed_rpc_server.cpp
  thrift-src/conn_mgr_pd_rpc_server.ipp
  thrift-src/mc_pd_rpc_server.ipp
  thrift-src/sswitch_pd_rpc_server.ipp
  ${ALL_THRIFT_CPP_FILES}
)
target_include_directories(bmpdfixed_thrift PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_BINARY_DIR}/gen-cpp
)
add_dependencies(bmpdfixed_thrift genthrift_pdfixed)

# Installation
install(FILES
  include/bm/pdfixed/pd_pre.h
  include/bm/pdfixed/pd_mirroring.h
  include/bm/pdfixed/pd_static.h
  include/bm/pdfixed/pd_common.h
  include/bm/pdfixed/pd_swap.h
  DESTINATION include/bm/pdfixed
)

install(FILES
  include/bm/pdfixed/int/pd_conn_mgr.h
  include/bm/pdfixed/int/pd_helpers.h
  include/bm/pdfixed/int/pd_notifications.h
  DESTINATION include/bm/pdfixed/int
)

install(FILES
  thrift-src/pdfixed_rpc_server.h
  DESTINATION include/bm/pdfixed/thrift-src
)

# FIXME: should res.thrift really be installed? Carried over from Makefile.am
install(FILES
  thrift/res.thrift
  DESTINATION include/bm/pdfixed/thrift
)

install(TARGETS bmpdfixed bmpdfixed_thrift
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
