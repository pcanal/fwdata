#target.symbol DataFormatsL1GlobalMuonTrigger
include_directories(${CMAKE_SOURCE_DIR}
${Boost_INCLUDE_DIR}
${Boost_INCLUDE_DIR}/python
${FMT_INCLUDE_DIR}
${CMSMD5_INCLUDE_DIR}
${Python_INCLUDE_DIR}
${ROOT_INCLUDE_DIR}
${TBB_INCLUDE_DIRS}
${TINYXML2_INCLUDE_DIR}
${UUID_INCLUDE_DIR}
/usr/include
)

cms_rootdict(DataFormatsL1GlobalMuonTrigger src/classes.h src/classes_def.xml)

add_rootdict_rules(DataFormatsL1GlobalMuonTrigger)

add_library(DataFormatsL1GlobalMuonTrigger SHARED ${DataFormatsL1GlobalMuonTrigger_EXTRA_SOURCES}
  src/L1MuGMTCand.cc
  src/L1MuGMTExtendedCand.cc
  src/L1MuGMTReadoutCollection.cc
  src/L1MuGMTReadoutRecord.cc
  src/L1MuRegionalCand.cc
)

set_target_properties(DataFormatsL1GlobalMuonTrigger PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

 add_dependencies( ALL_LIBS DataFormatsL1GlobalMuonTrigger)

install( TARGETS DataFormatsL1GlobalMuonTrigger DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR} EXPORT stitched::)

target_link_libraries(DataFormatsL1GlobalMuonTrigger PUBLIC
ROOT::Core
DataFormatsCommon
FWCoreMessageLogger
)

