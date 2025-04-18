#target.symbol FWCoreUtilities
include_directories(${CMAKE_SOURCE_DIR}
${Boost_INCLUDE_DIR}
${Boost_INCLUDE_DIR}/python
${FMT_INCLUDE_DIR}
${CMSMD5_INCLUDE_DIR}
${Python_INCLUDE_DIR}
${TBB_INCLUDE_DIRS}
${UUID_INCLUDE_DIR}
/usr/include
)

add_library(FWCoreUtilities SHARED ${FWCoreUtilities_EXTRA_SOURCES}
  src/Adler32Calculator.cc
  src/CPUServiceBase.cc
  src/CPUTimer.cc
  src/CRC32Calculator.cc
  src/ConvertException.cc
  src/DebugMacros.cc
  src/Digest.cc
  src/EDMException.cc
  src/ESInputTag.cc
  src/Exception.cc
  src/ExceptionCollector.cc
  src/ExceptionPropagate.cc
  src/FriendlyName.cc
  src/GlobalIdentifier.cc
  src/Guid.cc
  src/InputTag.cc
  src/LuminosityBlockIndex.cc
  src/MallocOpts.cc
  src/OStreamColumn.cc
  src/Parse.cc
  src/Presence.cc
  src/ProcessBlockIndex.cc
  src/RandomNumberGenerator.cc
  src/RegexMatch.cc
  src/ReleaseVersion.cc
  src/RunIndex.cc
  src/SingleConsumerQ.cc
  src/TestHelper.cc
  src/TimeOfDay.cc
  src/TimingServiceBase.cc
  src/TypeDemangler.cc
  src/TypeID.cc
  src/UnixSignalHandlers.cc
  src/WallclockTimer.cc
  src/WrappedClassName.cc
  src/resolveSymbolicLinks.cc
  src/stemFromPath.cc
  src/typelookup.cc
)

set_target_properties(FWCoreUtilities PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

add_dependencies( ALL_LIBS FWCoreUtilities)

install( TARGETS FWCoreUtilities DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR} EXPORT stitched::)

target_link_libraries(FWCoreUtilities PUBLIC
${CMSMD5_LIBRARY}
${TBB_LIBRARIES}
${UUID_LIBRARIES}
${Boost_REGEX_LIBRARY_RELEASE}
${CRYPT}
${DL}
${NSL}
${RT}
fmt::fmt
stdc++fs
)

