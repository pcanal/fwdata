#target.symbol DataFormatsL1GlobalCaloTrigger
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

cms_rootdict(DataFormatsL1GlobalCaloTrigger src/classes.h src/classes_def.xml)

add_rootdict_rules(DataFormatsL1GlobalCaloTrigger)

add_library(DataFormatsL1GlobalCaloTrigger SHARED ${DataFormatsL1GlobalCaloTrigger_EXTRA_SOURCES}
  src/L1GctCand.cc
  src/L1GctEmCand.cc
  src/L1GctEtHad.cc
  src/L1GctEtMiss.cc
  src/L1GctEtTotal.cc
  src/L1GctFibreWord.cc
  src/L1GctHFBitCounts.cc
  src/L1GctHFRingEtSums.cc
  src/L1GctHtMiss.cc
  src/L1GctInternEmCand.cc
  src/L1GctInternEtSum.cc
  src/L1GctInternHFData.cc
  src/L1GctInternHtMiss.cc
  src/L1GctInternJetData.cc
  src/L1GctJetCand.cc
  src/L1GctJetCounts.cc
  src/L1GctStaticParameters.cc
)

set_target_properties(DataFormatsL1GlobalCaloTrigger PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

 add_dependencies( ALL_LIBS DataFormatsL1GlobalCaloTrigger)

install( TARGETS DataFormatsL1GlobalCaloTrigger DESTINATION ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR} EXPORT stitched::)

target_link_libraries(DataFormatsL1GlobalCaloTrigger PUBLIC
ROOT::Core
DataFormatsCommon
DataFormatsL1CaloTrigger
${CRYPT}
${DL}
${NSL}
${RT}
)

