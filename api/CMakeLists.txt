# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

add_library(opentelemetry_api INTERFACE)
target_include_directories(
  opentelemetry_api
  INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
            "$<INSTALL_INTERFACE:include>")

set_target_properties(opentelemetry_api PROPERTIES EXPORT_NAME api)

if(OPENTELEMETRY_INSTALL)
  install(
    TARGETS opentelemetry_api
    EXPORT "${PROJECT_NAME}-target"
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

  install(
    DIRECTORY include/opentelemetry
    DESTINATION include
    FILES_MATCHING
    PATTERN "*.h"
    PATTERN "metrics" EXCLUDE)

  install(
    DIRECTORY include/opentelemetry
    DESTINATION include
    FILES_MATCHING
    PATTERN "*.h")

  unset(TARGET_DEPS)
endif()

if(BUILD_TESTING)
  add_subdirectory(test)
endif()

if(WITH_NO_DEPRECATED_CODE)
  target_compile_definitions(opentelemetry_api
                             INTERFACE OPENTELEMETRY_NO_DEPRECATED_CODE)
endif()

if(WITH_ABSEIL)
  target_compile_definitions(opentelemetry_api INTERFACE HAVE_ABSEIL)
  target_link_libraries(
    opentelemetry_api INTERFACE absl::bad_variant_access absl::any absl::base
                                absl::bits absl::city)
  list(APPEND TARGET_DEPS "absl_bad_variant_access" "absl_any absl_base"
       "absl_bits" "absl_city")
endif()

if(WITH_STL)
  message("Building with standard library types...")
  target_compile_definitions(opentelemetry_api INTERFACE HAVE_CPP_STDLIB)
else()
  message("Building with nostd types...")
endif()

if(WITH_GSL)
  target_compile_definitions(opentelemetry_api INTERFACE HAVE_GSL)

  # Guidelines Support Library path. Used if we are not on not get C++20.
  #
  find_package(Microsoft.GSL QUIET)
  if(TARGET Microsoft.GSL::GSL)
    target_link_libraries(opentelemetry_api INTERFACE Microsoft.GSL::GSL)
    list(APPEND TARGET_DEPS "gsl")
  else()
    set(GSL_DIR third_party/ms-gsl)
    target_include_directories(
      opentelemetry_api INTERFACE "$<BUILD_INTERFACE:${GSL_DIR}/include>")
  endif()
endif()

if(WITH_NO_GETENV)
  target_compile_definitions(opentelemetry_api INTERFACE NO_GETENV)
endif()

if(WIN32)
  target_compile_definitions(opentelemetry_api INTERFACE NOMINMAX)
  if(WITH_ETW)
    target_compile_definitions(opentelemetry_api INTERFACE HAVE_MSGPACK)
  endif()
endif()

if(WITH_ASYNC_EXPORT_PREVIEW)
  target_compile_definitions(opentelemetry_api INTERFACE ENABLE_ASYNC_EXPORT)
endif()

if(WITH_REMOVE_METER_PREVIEW)
  target_compile_definitions(opentelemetry_api
                             INTERFACE ENABLE_REMOVE_METER_PREVIEW)
endif()

target_compile_definitions(
  opentelemetry_api
  INTERFACE OPENTELEMETRY_ABI_VERSION_NO=${OPENTELEMETRY_ABI_VERSION_NO})

# A better place should be in sdk, not api
if(WITH_OTLP_HTTP_SSL_PREVIEW)
  target_compile_definitions(opentelemetry_api
                             INTERFACE ENABLE_OTLP_HTTP_SSL_PREVIEW)
  target_compile_definitions(opentelemetry_api
                             INTERFACE ENABLE_HTTP_SSL_PREVIEW)

  if(WITH_OTLP_HTTP_SSL_TLS_PREVIEW)
    target_compile_definitions(opentelemetry_api
                               INTERFACE ENABLE_OTLP_HTTP_SSL_TLS_PREVIEW)
    target_compile_definitions(opentelemetry_api
                               INTERFACE ENABLE_HTTP_SSL_TLS_PREVIEW)
  endif()
endif()

if(WITH_OTLP_GRPC_SSL_MTLS_PREVIEW)
  target_compile_definitions(opentelemetry_api
                             INTERFACE ENABLE_OTLP_GRPC_SSL_MTLS_PREVIEW)
endif()

if(WITH_METRICS_EXEMPLAR_PREVIEW)
  target_compile_definitions(opentelemetry_api
                             INTERFACE ENABLE_METRICS_EXEMPLAR_PREVIEW)
endif()

include(${PROJECT_SOURCE_DIR}/cmake/pkgconfig.cmake)

if(OPENTELEMETRY_INSTALL)
  opentelemetry_add_pkgconfig(
    api "OpenTelemetry API"
    "A header-only library to support instrumentation with OpenTelemetry."
    "${TARGET_DEPS}")
endif()
