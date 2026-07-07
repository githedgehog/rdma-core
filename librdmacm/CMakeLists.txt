publish_headers(rdma
  rdma_cma.h
  rdma_cma_abi.h
  rdma_verbs.h
  rsocket.h
  )
publish_headers(infiniband
  acm.h
  ib.h
  )

rdma_library(rdmacm librdmacm.map
  # See Documentation/versioning.md
  1 1.4.${PACKAGE_VERSION}
  acm.c
  addrinfo.c
  cma.c
  indexer.c
  rsocket.c
  )
target_link_libraries(rdmacm LINK_PUBLIC ibverbs)
target_link_libraries(rdmacm LINK_PRIVATE
  ${NL_LIBRARIES}
  ${CMAKE_THREAD_LIBS_INIT}
  ${RT_LIBRARIES}
  )

# The preload library is a bit special, it needs to be open coded
# Since it is a LD_PRELOAD it has no soname, and is installed in sub dir
# The version script is generated from librspreload.map.in so the fcntl64/sendfile64
# entries can be gated on RDMA_PRELOAD_WRAP_LFS64 (see top-level CMakeLists.txt).
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/librspreload.map.in"
  "${CMAKE_CURRENT_BINARY_DIR}/librspreload.map"
  @ONLY)
rdma_rspreload_module(rspreload "${CMAKE_CURRENT_BINARY_DIR}/librspreload.map"
  preload.c
  indexer.c
  )
target_link_libraries(rspreload LINK_PRIVATE
  rdmacm
  ${CMAKE_THREAD_LIBS_INIT}
  ${CMAKE_DL_LIBS}
)
install(TARGETS rspreload DESTINATION "${CMAKE_INSTALL_LIBDIR}/rsocket/")

# These are for compat with old packaging, these name should not be used.
# FIXME: Maybe we can get rid of them?
rdma_install_symlink("librspreload.so" "${CMAKE_INSTALL_LIBDIR}/rsocket/librspreload.so.1")
rdma_install_symlink("librspreload.so" "${CMAKE_INSTALL_LIBDIR}/rsocket/librspreload.so.1.0.0")

if (ENABLE_STATIC)
  if (NOT NL_KIND EQUAL 0)
    set(REQUIRES "libnl-3.0, libnl-route-3.0, ")
  endif()
endif()
rdma_pkg_config("rdmacm" "${REQUIRES}libibverbs" "${CMAKE_THREAD_LIBS_INIT}")
