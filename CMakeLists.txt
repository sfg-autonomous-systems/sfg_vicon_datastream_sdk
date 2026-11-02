cmake_minimum_required(VERSION 3.12)
project(sfg_vicon_datastream_sdk VERSION 1.13.0)

# ########################################################################################################
# Find dependencies.                                                                                     #
# ########################################################################################################
find_package(Threads REQUIRED)

# ########################################################################################################
# Add libraries.                                                                                         #
# ########################################################################################################
set(target_name "ViconDataStreamSDK_CPP")
set(sdk_directory "${CMAKE_CURRENT_SOURCE_DIR}/Vicon/CrossMarket")
set(stream_directory "${sdk_directory}/DataStream")
set(generated_directory "${CMAKE_CURRENT_BINARY_DIR}/generated")

configure_file(
  "cmake/ViconDataStreamSDKCoreVersion.h.in"
  "${generated_directory}/ViconDataStreamSDKCoreVersion.h"
)

# The SDK is always static and position independent, so that it can be linked into shared
# libraries without having to install anything alongside them.
add_library("${target_name}" STATIC
  "${sdk_directory}/StreamCommon/CGStreamAsyncReaderWriter.cpp"
  "${stream_directory}/ViconCGStreamClient/CGStreamReaderWriter.cpp"
  "${stream_directory}/ViconCGStreamClient/ViconCGStreamBayer.cpp"
  "${stream_directory}/ViconCGStreamClient/ViconCGStreamClient.cpp"
  "${stream_directory}/ViconCGStreamClientSDK/CGClient.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/CoreClient.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/CoreClientTimingLog.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/RetimerUtils.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/RetimingClient.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/RetimingCore.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/SegmentPoseReader.cpp"
  "${stream_directory}/ViconDataStreamSDKCore/WirelessConfiguration.cpp"
  "${stream_directory}/ViconDataStreamSDKCoreUtils/AxisMapping.cpp"
  "${stream_directory}/ViconDataStreamSDKCoreUtils/ClientUtils.cpp"
  "${stream_directory}/ViconDataStreamSDK_CPP/DataStreamClient.cpp"
  "${stream_directory}/ViconDataStreamSDK_CPP/DataStreamRetimingClient.cpp"
)
target_include_directories("${target_name}" SYSTEM
  PUBLIC
  "${stream_directory}/ViconDataStreamSDK_CPP"
  PRIVATE
  "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/AsioStandalone/AsioStandalone-1.18.1/include"
  "${generated_directory}"
  "${sdk_directory}"
  "${stream_directory}"
)
target_compile_definitions("${target_name}" PRIVATE ASIO_STANDALONE=1)
target_compile_features("${target_name}" PRIVATE cxx_std_20)
target_link_libraries("${target_name}" PUBLIC ${CMAKE_THREAD_LIBS_INIT})
set_target_properties("${target_name}" PROPERTIES
  POSITION_INDEPENDENT_CODE ON
  CXX_VISIBILITY_PRESET hidden
  VISIBILITY_INLINES_HIDDEN ON
)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # Vicon's makefiles build with -fno-strict-aliasing, so the SDK may rely on it.
  # -w keeps warning flags inherited from a parent project from reporting on code we don't own.
  target_compile_options("${target_name}" PRIVATE -fno-strict-aliasing -w)
endif()
