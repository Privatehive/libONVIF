include(GenerateExportHeader)
include(CMakePackageConfigHelpers)
find_package(Qt6 COMPONENTS Core REQUIRED)
if (ENABLE_CODESYNTHESIS)
    find_program(Gsoap_wsdl2h_PATH NAMES wsdl2h PATH_SUFFIXES "bin")
    find_program(Gsoap_soapcpp2_PATH NAMES soapcpp2 PATH_SUFFIXES "bin")
    find_path(Gsoap_import_DIR NAMES wsdd5.h PATH_SUFFIXES "gsoap/import" "share/gsoap/import" "share/gsoap")
    find_path(Gsoap_plugin_DIR NAMES wsseapi.h PATH_SUFFIXES "gsoap/plugin" "share/gsoap/plugin" "share/gsoap")
    find_path(Gsoap_custom_DIR NAMES struct_timeval.h PATH_SUFFIXES "gsoap/custom" "share/gsoap/custom" "share/gsoap")
    find_file(Gsoap_stdsoap2cpp NAMES stdsoap2.cpp PATH_SUFFIXES "gsoap" REQUIRED)
    find_file(Gsoap_stdsoap2h NAMES stdsoap2.h PATH_SUFFIXES "gsoap" REQUIRED)
    find_file(Gsoap_domcpp NAMES dom.cpp PATH_SUFFIXES "gsoap" REQUIRED)
endif ()

set(BUILD_SHARED_LIBS true CACHE BOOL "Build shared or static lib?")
set(ENABLE_CODESYNTHESIS false CACHE BOOL "Run wsdl2h and soapcpp2?")
set(ENABLE_SSL false CACHE BOOL "Enable ssl support?")

set(HAS_OPENSSL OFF)
if (ENABLE_SSL)
    include(FindOpenSSL)
    if (NOT OPENSSL_FOUND)
        message(FATAL_ERROR "OpenSSL not found")
    endif ()
    set(HAS_OPENSSL ON)
endif ()

include_directories("${CMAKE_CURRENT_BINARY_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}/generated" "${CMAKE_CURRENT_SOURCE_DIR}/gsoap")
if (HAS_OPENSSL)
    add_definitions(-DWITH_OPENSSL)
else ()
    message(STATUS "SSL/TLS disabled because of missing OpenSSL library")
endif ()

if (WIN32)
    add_definitions(-DOS_WIN32)
endif ()

if (MSVC)
    add_definitions(-DNOMINMAX)
endif ()

if (WIN32 AND BUILD_SHARED_LIBS)
    add_definitions(-DSOAP_STD_EXPORTS)
endif ()

# -------------------------------GSOAP synthesis-------------------------------

set(GSOAP_DIRS "${Gsoap_import_DIR}/../:${Gsoap_import_DIR}:${Gsoap_custom_DIR}:${CMAKE_CURRENT_SOURCE_DIR}/gsoap")
if (ENABLE_CODESYNTHESIS)
    message(STATUS "Using gsoap dirs: ${GSOAP_DIRS}")
    add_custom_target(Synthesis
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsseapi.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsseapi.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsseapi-lite.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsseapi-lite.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/mecevp.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/mecevp.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsaapi.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/wsaapi.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/smdevp.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/smdevp.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/httpda.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_plugin_DIR}/httpda.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap

            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/qstring.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/qstring.cpp ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/qdatetime.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/qdatetime.cpp ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/duration.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_custom_DIR}/duration.c ${CMAKE_CURRENT_SOURCE_DIR}/gsoap

            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_import_DIR}/stlvector.h ${CMAKE_CURRENT_SOURCE_DIR}/gsoap

            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_stdsoap2cpp} ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_stdsoap2h} ${CMAKE_CURRENT_SOURCE_DIR}/gsoap
            COMMAND ${CMAKE_COMMAND} -E copy ${Gsoap_domcpp} ${CMAKE_CURRENT_SOURCE_DIR}/gsoap

            COMMAND ${Gsoap_wsdl2h_PATH} -c++11 -z7 -d -p -j -t typemap.dat -o generated.h
            http://www.onvif.org/onvif/ver10/analyticsdevice.wsdl
            http://www.onvif.org/onvif/ver10/replay.wsdl
            http://www.onvif.org/onvif/ver10/search.wsdl
            http://www.onvif.org/onvif/ver10/recording.wsdl
            http://www.onvif.org/onvif/ver10/receiver.wsdl
            http://www.onvif.org/onvif/ver20/ptz/wsdl/ptz.wsdl
            http://www.onvif.org/onvif/ver10/media/wsdl/media.wsdl
            http://www.onvif.org/onvif/ver20/media/wsdl/media.wsdl
            http://www.onvif.org/onvif/ver20/imaging/wsdl/imaging.wsdl
            http://www.onvif.org/onvif/ver10/display.wsdl
            http://www.onvif.org/onvif/ver10/events/wsdl/event.wsdl
            http://www.onvif.org/onvif/ver10/device/wsdl/devicemgmt.wsdl
            http://www.onvif.org/onvif/ver20/analytics/wsdl/analytics.wsdl
            COMMAND ${Gsoap_soapcpp2_PATH} -Ed -Ec -c++11 -z4 -a -j -L -x -n -d ${CMAKE_CURRENT_SOURCE_DIR}/generated -I ${GSOAP_DIRS} generated.h
            COMMAND $<TARGET_FILE:headerParserTool> generated.h
            COMMAND ${CMAKE_COMMAND} -E copy GsoapResolver.h ${CMAKE_CURRENT_SOURCE_DIR}/generated
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/wsdl VERBATIM)

    add_dependencies(Synthesis headerParserTool)
endif ()

# -------------------------------Header parser-------------------------------

set(src_files
        headerParser/main.cpp
)

add_executable(headerParserTool ${src_files})

# -------------------------------ONVIF Common-------------------------------
set(src_files
        generated/soapC.cpp
        gsoap/duration.c
        gsoap/qdatetime.cpp
        gsoap/AbsoluteOrRelativeTimeType.cpp
        gsoap/qstring.cpp
        gsoap/stdsoap2.cpp
        gsoap/dom.cpp
        gsoap/wsaapi.c
        gsoap/qstringCompare.cpp
        SoapCtx.cpp
        SoapAuthHandler.cpp
        Client.cpp
        SoapHelper.cpp
)

if (HAS_OPENSSL)
    set(src_files ${src_files} gsoap/smdevp.c gsoap/httpda.c gsoap/mecevp.c gsoap/wsseapi.c)
else ()
    set(src_files ${src_files} gsoap/wsseapi-lite.c)
endif ()

# private header
set(private_header_files
        gsoap/stlvector.h
)

# public header
set(public_header_files
        generated/soapStub.h
        generated/soapH.h
        gsoap/stdsoap2.h
        gsoap/wsaapi.h
        gsoap/AbsoluteOrRelativeTime.h
        gsoap/qstringCompare.h
        SoapCtx.h
        SoapAuthHandler.h
        Request.h
        Response.h
        Client.h
        global.h
        SoapHelper.h
)

if (HAS_OPENSSL)
    set(public_header_files ${public_header_files} gsoap/smdevp.h gsoap/httpda.h gsoap/threads.h gsoap/mecevp.h gsoap/wsseapi.h)
else ()
    set(public_header_files ${public_header_files} gsoap/wsseapi-lite.h)
endif ()

set_source_files_properties(${src_files} PROPERTIES LANGUAGE CXX)

add_library(onvifcommon ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifcommon EXPORT_FILE_NAME OnvifCommonExport.h)
target_include_directories(onvifcommon
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> $<INSTALL_INTERFACE:include>
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/gsoap> $<INSTALL_INTERFACE:include>
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/generated> $<INSTALL_INTERFACE:include>
        PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)
target_link_libraries(onvifcommon PUBLIC Qt6::Core)
target_compile_definitions(onvifcommon PUBLIC WITH_SELF_PIPE PUBLIC WITH_DOM)
if (HAS_OPENSSL)
    target_compile_definitions(onvifcommon PUBLIC WITH_OPENSSL)
    target_link_libraries(onvifcommon PUBLIC OpenSSL::SSL PUBLIC OpenSSL::Crypto)
endif ()
if (WIN32)
    target_link_libraries(onvifcommon PUBLIC ws2_32)
    if (NOT BUILD_SHARED_LIBS)
        target_link_libraries(onvifcommon PUBLIC version PUBLIC Winmm PUBLIC Netapi32 PUBLIC Userenv)
        if (HAS_OPENSSL)
            target_link_libraries(onvifcommon PUBLIC crypt32)
        endif ()
    endif ()
endif ()
target_compile_features(onvifcommon PUBLIC cxx_strong_enums cxx_auto_type cxx_nullptr cxx_long_long_type cxx_explicit_conversions)
set_target_properties(onvifcommon PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifCommonExport.h")

# -------------------------------ONVIF Analytics-------------------------------
set(src_files
        generated/soapAnalyticsDeviceBindingProxy.cpp
        #generated/soapAnalyticsDeviceBindingService.cpp
        OnvifAnalyticsClient.cpp
)

# private header
set(private_header_files
        generated/soapAnalyticsDeviceBindingProxy.h
        #generated/soapAnalyticsDeviceBindingService.h
)

# public header
set(public_header_files
        OnvifAnalyticsClient.h
)

add_library(onvifanalytics ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifanalytics EXPORT_FILE_NAME OnvifAnalyticsExport.h)
target_link_libraries(onvifanalytics onvifcommon)
set_target_properties(onvifanalytics PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifAnalyticsExport.h")

# -------------------------------ONVIF Device-------------------------------
set(src_files
        generated/soapDeviceBindingProxy.cpp
        #generated/soapDeviceBindingService.cpp
        OnvifDeviceClient.cpp
)

# private header
set(private_header_files
        generated/soapDeviceBindingProxy.h
        #generated/soapDeviceBindingService.h
)

# public header
set(public_header_files
        OnvifDeviceClient.h
)

add_library(onvifdevice ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifdevice EXPORT_FILE_NAME OnvifDeviceExport.h)
target_link_libraries(onvifdevice onvifcommon)
set_target_properties(onvifdevice PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifDeviceExport.h")

# -------------------------------ONVIF Display-------------------------------
set(src_files
        generated/soapDisplayBindingProxy.cpp
        #generated/soapDisplayBindingService.cpp
        OnvifDisplayClient.cpp
)

# private header
set(private_header_files
        generated/soapDisplayBindingProxy.h
        #generated/soapDisplayBindingService.h
)

# public header
set(public_header_files
        OnvifDisplayClient.h
)

add_library(onvifdisplay ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifdisplay EXPORT_FILE_NAME OnvifDisplayExport.h)
target_link_libraries(onvifdisplay onvifcommon)
set_target_properties(onvifdisplay PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifDisplayExport.h")

# -------------------------------ONVIF Imaging-------------------------------
set(src_files
        generated/soapImagingBindingProxy.cpp
        #generated/soapImagingBindingService.cpp
        OnvifImagingClient.cpp
)

# private header
set(private_header_files
        generated/soapImagingBindingProxy.h
        #generated/soapImagingBindingService.h
)

# public header
set(public_header_files
        OnvifImagingClient.h
)

add_library(onvifimaging ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifimaging EXPORT_FILE_NAME OnvifImagingExport.h)
target_link_libraries(onvifimaging onvifcommon)
set_target_properties(onvifimaging PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifImagingExport.h")

# -------------------------------ONVIF Media-------------------------------
set(src_files
        generated/soapMediaBindingProxy.cpp
        #generated/soapMediaBindingService.cpp
        OnvifMediaClient.cpp
)

# private header
set(private_header_files
        generated/soapMediaBindingProxy.h
        #generated/soapMediaBindingService.h
)

# public header
set(public_header_files
        OnvifMediaClient.h
)

add_library(onvifmedia ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifmedia EXPORT_FILE_NAME OnvifMediaExport.h)
target_link_libraries(onvifmedia onvifcommon)
set_target_properties(onvifmedia PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifMediaExport.h")

# -------------------------------ONVIF Media2-------------------------------
set(src_files
        generated/soapMedia2BindingProxy.cpp
        #generated/soapMedia2BindingService.cpp
        OnvifMedia2Client.cpp
)

# private header
set(private_header_files
        generated/soapMedia2BindingProxy.h
        #generated/soapMedia2BindingService.h
)

# public header
set(public_header_files
        OnvifMedia2Client.h
)

add_library(onvifmedia2 ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifmedia2 EXPORT_FILE_NAME OnvifMedia2Export.h)
target_link_libraries(onvifmedia2 onvifcommon)
set_target_properties(onvifmedia2 PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifMedia2Export.h")

# -------------------------------ONVIF PTZ-------------------------------
set(src_files
        generated/soapPTZBindingProxy.cpp
        #generated/soapPTZBindingService.cpp
        OnvifPtzClient.cpp
)

# private header
set(private_header_files
        generated/soapPTZBindingProxy.h
        #generated/soapPTZBindingService.h
)

# public header
set(public_header_files
        OnvifPtzClient.h
)

add_library(onvifptz ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifptz EXPORT_FILE_NAME OnvifPtzExport.h)
target_link_libraries(onvifptz onvifcommon)
set_target_properties(onvifptz PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifPtzExport.h")

# -------------------------------ONVIF Event-------------------------------
set(src_files
        generated/soapPullPointSubscriptionBindingProxy.cpp
        #generated/soapPullPointSubscriptionBindingService.cpp
        OnvifEventClient.cpp
        OnvifPullPoint.cpp
        Topics.cpp
        NotificationFilter.cpp
)

# private header
set(private_header_files
        generated/soapPullPointSubscriptionBindingProxy.h
        #generated/soapPullPointSubscriptionBindingService.h
)

# public header
set(public_header_files
        OnvifEventClient.h
        OnvifPullPoint.h
        Topics.h
        NotificationFilter.h
)

add_library(onvifevent ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifevent EXPORT_FILE_NAME OnvifEventExport.h)
target_link_libraries(onvifevent onvifcommon)
set_target_properties(onvifevent PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifEventExport.h")

# -------------------------------ONVIF Receiver-------------------------------
set(src_files
        generated/soapReceiverBindingProxy.cpp
        #generated/soapReceiverBindingService.cpp
        OnvifReceiverClient.cpp
)

# private header
set(private_header_files
        generated/soapReceiverBindingProxy.h
        #generated/soapReceiverBindingService.h
)

# public header
set(public_header_files
        OnvifReceiverClient.h
)

add_library(onvifreceiver ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifreceiver EXPORT_FILE_NAME OnvifReceiverExport.h)
target_link_libraries(onvifreceiver onvifcommon)
set_target_properties(onvifreceiver PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifReceiverExport.h")

# -------------------------------ONVIF Recording-------------------------------
set(src_files
        generated/soapRecordingBindingProxy.cpp
        #generated/soapRecordingBindingService.cpp
        OnvifRecordingClient.cpp
)

# private header
set(private_header_files
        generated/soapRecordingBindingProxy.h
        #generated/soapRecordingBindingService.h
)

# public header
set(public_header_files
        OnvifRecordingClient.h
)

add_library(onvifrecording ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifrecording EXPORT_FILE_NAME OnvifRecordingExport.h)
target_link_libraries(onvifrecording onvifcommon)
set_target_properties(onvifrecording PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifRecordingExport.h")

# -------------------------------ONVIF Discovery-------------------------------
set(src_files
        # generated/soapRemoteDiscoveryBindingProxy.cpp
        # generated/soapRemoteDiscoveryBindingService.cpp
        # gsoap/wsddapi.c
        OnvifDiscoveryClient.cpp
        # OnvifDiscover.cpp
        OnvifDiscovery.cpp
        DiscoveryMatch.cpp
        generated/soapwsddProxy.cpp
)

set_source_files_properties(${src_files} PROPERTIES LANGUAGE CXX)

# private header
set(private_header_files
        # generated/soapRemoteDiscoveryBindingProxy.h
        # generated/soapRemoteDiscoveryBindingService.h
)

# public header
set(public_header_files
        OnvifDiscoveryClient.h
        DeleterDuplicator.h
        # OnvifDiscover.h
        generated/soapwsddProxy.h
        OnvifDiscovery.h
        DiscoveryMatch.h
)

add_library(onvifdiscovery ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifdiscovery EXPORT_FILE_NAME OnvifDiscoveryExport.h)
target_link_libraries(onvifdiscovery onvifcommon)
set_target_properties(onvifdiscovery PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifDiscoveryExport.h")

# -------------------------------ONVIF Replay-------------------------------
set(src_files
        generated/soapReplayBindingProxy.cpp
        #generated/soapReplayBindingService.cpp
        OnvifReplayClient.cpp
)

# private header
set(private_header_files
        generated/soapReplayBindingProxy.h
        #generated/soapReplayBindingService.h
)

# public header
set(public_header_files
        OnvifReplayClient.h
)

add_library(onvifreplay ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifreplay EXPORT_FILE_NAME OnvifReplayExport.h)
target_link_libraries(onvifreplay onvifcommon)
set_target_properties(onvifreplay PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifReplayExport.h")

# -------------------------------Device Probe-------------------------------
set(src_files
        DeviceProbe.cpp
)

# private header
set(private_header_files
)

# public header
set(public_header_files
        DeviceProbe.h
)

add_library(onvifprobe ${src_files} ${private_header_files} ${public_header_files})
generate_export_header(onvifprobe EXPORT_FILE_NAME OnvifProbeExport.h)
target_link_libraries(onvifprobe onvifcommon onvifdevice)
set_target_properties(onvifprobe PROPERTIES PUBLIC_HEADER "${public_header_files};${CMAKE_CURRENT_BINARY_DIR}/OnvifProbeExport.h")

# -------------------------------------------------------------------------

# info tool
set(src_files
        main.cpp
        CmdLineParser.cpp
        OnvifDevice.cpp
)

# private header
set(private_header_files
        CmdLineParser.h
        OnvifDevice.h
)

add_executable(onvifinfo ${src_files} ${private_header_files})
target_link_libraries(onvifinfo onvifcommon onvifdevice onvifdiscovery onvifreplay onvifprobe onvifrecording onvifreceiver onvifptz onvifmedia onvifmedia2 onvifimaging onvifevent onvifdisplay onvifanalytics)

if (MSVC)
    set_target_properties(onvifinfo onvifcommon onvifdevice onvifdiscovery onvifreplay onvifprobe onvifrecording onvifreceiver onvifptz onvifmedia onvifmedia2 onvifimaging onvifevent onvifdisplay onvifanalytics PROPERTIES COMPILE_FLAGS "/bigobj")
endif ()

if (ANDROID)
    target_link_libraries(onvifinfo log.so z.so)
endif ()

set(install_root "${CMAKE_INSTALL_PREFIX}")
set(install_include_dir "${install_root}/include")
set(install_runtime_dir "${install_root}/bin")
set(install_lib_dir "${install_root}/lib")
set(install_cmake_dir "${install_root}/lib/cmake")

set(INCLUDE_INSTALL_DIR "include")
set(CMAKE_INSTALL_DIR "cmake")
set(BIN_INSTALL_DIR "bin")
set(LIB_INSTALL_DIR "lib")

configure_package_config_file("${PROJECT_SOURCE_DIR}/CMake/libONVIFConfig.cmake.in" "${PROJECT_BINARY_DIR}/libONVIFConfig.cmake" INSTALL_DESTINATION cmake
        PATH_VARS INCLUDE_INSTALL_DIR CMAKE_INSTALL_DIR BIN_INSTALL_DIR LIB_INSTALL_DIR)

write_basic_package_version_file("${PROJECT_BINARY_DIR}/libONVIFVersion.cmake" VERSION "${info.version.major}.${info.version.minor}.${info.version.patch}" COMPATIBILITY SameMajorVersion)

# add the install target
install(TARGETS
        onvifinfo
        onvifcommon
        onvifdevice
        onvifdiscovery
        onvifreplay
        onvifprobe
        onvifrecording
        onvifreceiver
        onvifptz
        onvifmedia
        onvifmedia2
        onvifimaging
        onvifevent
        onvifdisplay
        onvifanalytics
        EXPORT libONVIFTargets RUNTIME DESTINATION bin ARCHIVE DESTINATION lib LIBRARY DESTINATION lib PUBLIC_HEADER DESTINATION include BUNDLE DESTINATION .
)

install(EXPORT libONVIFTargets DESTINATION cmake)

install(FILES "${PROJECT_BINARY_DIR}/libONVIFConfig.cmake" "${PROJECT_BINARY_DIR}/libONVIFVersion.cmake" DESTINATION cmake)
