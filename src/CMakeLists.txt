add_executable(bpftrace
  attached_probe.cpp
  bpftrace.cpp
  clang_parser.cpp
  driver.cpp
  fake_map.cpp
  main.cpp
  map.cpp
  mapkey.cpp
  printf.cpp
  resolve_cgroupid.cpp
  tracepoint_format_parser.cpp
  types.cpp
  utils.cpp
  list.cpp
)

if(HAVE_NAME_TO_HANDLE_AT)
  target_compile_definitions(bpftrace PRIVATE HAVE_NAME_TO_HANDLE_AT=1)
endif(HAVE_NAME_TO_HANDLE_AT)
if(HAVE_BCC_PROG_LOAD)
  target_compile_definitions(bpftrace PRIVATE HAVE_BCC_PROG_LOAD)
endif(HAVE_BCC_PROG_LOAD)
if(HAVE_BCC_CREATE_MAP)
  target_compile_definitions(bpftrace PRIVATE HAVE_BCC_CREATE_MAP)
endif(HAVE_BCC_CREATE_MAP)
if(HAVE_GET_CURRENT_CGROUP_ID)
  target_compile_definitions(bpftrace PRIVATE HAVE_GET_CURRENT_CGROUP_ID)
endif(HAVE_GET_CURRENT_CGROUP_ID)
target_link_libraries(bpftrace arch ast parser resources)

if (STATIC_LINKING)
  target_link_libraries(bpftrace ${LIBBCC_LIBRARIES})
  target_link_libraries(bpftrace ${LIBBPF_LIBRARY_STATIC})
else()
  target_link_libraries(bpftrace ${LIBBCC_LIBRARIES})
endif()
target_link_libraries(bpftrace ${LIBELF_LIBRARIES})

install(TARGETS bpftrace DESTINATION bin)

set(KERNEL_HEADERS_DIR "" CACHE PATH "Hard-code kernel headers directory")
if (KERNEL_HEADERS_DIR)
MESSAGE(STATUS "Using KERNEL_HEADERS_DIR=${KERNEL_HEADERS_DIR}")
target_compile_definitions(bpftrace PUBLIC KERNEL_HEADERS_DIR="${KERNEL_HEADERS_DIR}")
endif()

execute_process(
  COMMAND git describe --abbrev=4 --dirty --always --tags
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE BPFTRACE_VERSION
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

add_definitions("-DBPFTRACE_VERSION=\"${BPFTRACE_VERSION}\"")
