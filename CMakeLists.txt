
cmake_minimum_required(VERSION 3.12)
project(Framework VERSION 0.0.0 LANGUAGES C CXX)

if(BUILDTEST)
  include(CTest)
  enable_testing()
endif()
if(CMAKE_CXX_COMPILER_ID EQUAL Clang OR CMAKE_COMPILER_IS_GNUCC  OR CMAKE_COMPILER_IS_GNUCXX)
  if(UNIX AND NOT APPLE)
    SET( CMAKE_EXE_LINKER_FLAGS  "${CMAKE_EXE_LINKER_FLAGS} -z defs")
    SET( CMAKE_MODULE_LINKER_FLAGS  "${CMAKE_MODULE_LINKER_FLAGS} -z defs")
    SET( CMAKE_SHARED_LINKER_FLAGS  "${CMAKE_SHARED_LINKER_FLAGS} -z defs")
  endif()
endif()
cmake_policy( SET CMP0082 NEW)
set(CMAKE_CXX_STANDARD 17)
find_package(ROOT)
include(GNUInstallDirs)
set(CMAKE_INSTALL_LIBDIR "lib")
include(${ROOT_USE_FILE})
set(Boost_NO_BOOST_CMAKE ON)
set(Boost_NO_SYSTEM_PATHS ON)
find_package(Boost COMPONENTS filesystem thread iostreams regex serialization system program_options)
add_compile_definitions(BOOST_SPIRIT_THREADSAFE;PHOENIX_THREADSAFE;BOOST_MATH_DISABLE_STD_FPCLASSIFY;BOOST_UUID_RANDOM_PROVIDER_FORCE_POSIX;TBB_USE_GLIBCXX_VERSION=80400;TBB_SUPPRESS_DEPRECATED_MESSAGES)
find_package(Python COMPONENTS Development)
set(Python_INCLUDE_DIR ${Python_INCLUDE_DIRS})
set(Python_LIBRARY Python::Python)
find_package(Threads)
find_package(CMakeTools)
UseCMakeTools()
add_custom_target(ALL_LIBS ALL)
add_custom_target(ALL_DICTS ALL)
add_dependencies(ALL_LIBS ALL_DICTS)
add_custom_target(ALL_PLUGINS ALL)
add_dependencies(ALL_LIBS ALL_PLUGINS)
add_custom_target(ALL_TESTLIBS ALL)
add_dependencies(ALL_LIBS ALL_TESTLIBS)
add_custom_target(ALL_BINS ALL)
add_custom_target(ALL_TESTBINS ALL)
add_dependencies(ALL_BINS ALL_LIBS)
add_dependencies(ALL_TESTBINS ALL_LIBS)

set(testdir ${CMAKE_BINARY_DIR}/test)
set(pythondir ${CMAKE_BINARY_DIR}/python)
set(sourcedir ${CMAKE_BINARY_DIR}/src)
set(pcmfiles "")
set(rootmapfiles "")
file(MAKE_DIRECTORY ${testdir} )
file(MAKE_DIRECTORY ${sourcedir})
file(GLOB sources "${CMAKE_SOURCE_DIR}/*")
file(COPY ${sources} DESTINATION ${sourcedir} PATTERN ".git" EXCLUDE)
#add_custom_target(file_in_path
#COMMAND mkdir -p "${CMAKE_BINARY_DIR}/test/Geometry/TrackerSimData/data"
#COMMAND touch  "${CMAKE_BINARY_DIR}/test/Geometry/TrackerSimData/data/trackersens.xml"
#COMMAND touch "${CMAKE_BINARY_DIR}/test/tmp.py")
file(MAKE_DIRECTORY ${pythondir})
file(WRITE  "${pythondir}/__init__.py" "#autogenerated")

find_package(CLHEP)

find_package(CMSMD5)

find_package(HepMC)

find_package(OpenGL)

find_package(TBB)

find_package(TINYXML2)

find_package(UUID)

find_package(VDT)

find_package(fmt)

find_library(NSL nsl NAMES nsl-2.28 nsl-2.26 nsl-2.17 nsl)
find_library(RT rt)
find_library(CRYPT crypt)
find_library(DL dl)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-attributes")


 
macro(cms_rootdict name)
  if("${ARGN}" STREQUAL "")
    set(classes_h classes.h)
    set(classes_xml classes_def.xml)
    set(${name}_ROOTDICT ${name}_x)
  else()
    set(classes_h ${ARGV1})
    set(classes_xml ${ARGV2})
    if(${name}_ROOTDICT)
      MATH(EXPR ${name}_DICT_COUNT "${${name}_DICT_COUNT}+1")
      set(${name}_ROOTDICT ${name}_x${${name}_DICT_COUNT})
    else()
      set(${name}_DICT_COUNT 0)
      set(${name}_ROOTDICT ${name}_x)
    endif()
  endif()
  set(${name}_EXTRA_SOURCES ${${name}_EXTRA_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/${${name}_ROOTDICT}.cc)
  set(${name}_ROOTDICTS ${${name}_ROOTDICTS} ${${name}_ROOTDICT})
  set(ROOTDICT_${${name}_ROOTDICT} ${classes_h} ${classes_xml} ${CMAKE_CURRENT_BINARY_DIR}/${${name}_ROOTDICT}.cc)
  unset(clases_h)
  unset(classes_xml)
endmacro()

macro(add_rootdict_rules name)
  set(flags "")
  get_directory_property(defs COMPILE_DEFINITIONS)
  if(defs)
    foreach(d ${defs})
      set(flags ${flags} -D${d})
    endforeach()
  endif()
  get_directory_property(dirs INCLUDE_DIRECTORIES)
  if(dirs)
    foreach(inc ${dirs})
      set(flags ${flags} -I${inc})
    endforeach()
  endif()
  foreach(dict ${${name}_ROOTDICTS})
    list(GET ROOTDICT_${dict} 0 classes_h)
    list(GET ROOTDICT_${dict} 1 classes_xml)
    list(GET ROOTDICT_${dict} 2 classes_cc)
    get_filename_component(classname ${classes_h} NAME_WE)
    set_source_files_properties(${classes_cc} PROPERTIES COMPILE_FLAGS "-Os -Wno-unused-variable")
    add_custom_command(OUTPUT ${classes_cc}
                    COMMAND ${ROOT_genreflex_CMD} ${CMAKE_CURRENT_SOURCE_DIR}/${classes_h}
                    -s ${CMAKE_CURRENT_SOURCE_DIR}/${classes_xml}
                    -o ${classes_cc}
		    --rootmap=${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}/${dict}.rootmap
		    --rootmap-lib=${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}/lib${name}.so
		    --library ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}/lib${name}.so --multiDict
                    -DCMS_DICT_IMPL -D_REENTRANT -DGNUSOURCE -D__STRICT_ANSI__ ${FLAGS} -DCMSSW_REFLEX_DICT 
                    -I${CMAKE_SOURCE_DIR} ${flags}
                    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/${classes_h} ${CMAKE_CURRENT_SOURCE_DIR}/${classes_xml}
                    )
    list(APPEND rootmapfiles ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}/${dict}.rootmap)
    list(APPEND pcmfiles ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}/lib${dict}_${dict}_x_rdict.pcm)
  endforeach()
  unset(clases_h)
  unset(classes_xml)
  unset(classname)
  unset(FLAGS)
endmacro()

#add_custom_command(TARGET ALL_LIBS POST_BUILD
#  COMMAND $<TARGET_FILE:edmPluginRefresh> "plugin*.so" 
#  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/lib
#  DEPENDS edmPluginRefresh
#  BYPRODUCTS ${CMAKE_BINARY_DIR}/lib/.edmplugincache
#)

macro(add_script_rules target)
  file(GLOB scripts ${CMAKE_CURRENT_SOURCE_DIR}/scripts/*)
  if(scripts)
      foreach(script ${scripts})
          file(COPY ${script} DESTINATION ${CMAKE_BINARY_DIR}/bin)
          install(FILES ${script}  DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
      endforeach()
  endif()
  unset(scripts)
endmacro()


macro(edm_write_config target dir)
  set(cfidir ${CMAKE_BINARY_DIR}/cfipython/${dir})
  file(MAKE_DIRECTORY ${cfidir})
  add_custom_command( TARGET ${target} POST_BUILD
                    COMMAND $<TARGET_FILE:edmWriteConfigs> -p $<TARGET_FILE:${target}>
                    WORKING_DIRECTORY ${cfidir}
                    DEPENDS edmWriteConfigs ${target}
                    )
  unset(cfidir)
endmacro()

macro(edm_register_plugin target dir)
endmacro()

macro(add_python_rules target)
  set(pythonsrcdir ${sourcedir}/${target}/python)
  file(GLOB pyfiles ${pythonsrcdir}/*.py)
  file(GLOB tpyfiles ${pythonsrcdir}/test/*.py)
  set(pythondir ${CMAKE_BINARY_DIR}/python/${target})
  get_filename_component(pdir ${pythondir} DIRECTORY)
  if(pyfiles OR tpyfiles)
      file(MAKE_DIRECTORY ${pdir})
      file(RELATIVE_PATH reldir ${pdir} ${pythonsrcdir} )
      file(CREATE_LINK ${reldir}  ${pythondir} SYMBOLIC)
      set(dnames "")
      foreach(file ${tpyfiles})
          get_filename_component(dname ${file} DIRECTORY)
          list(APPEND dnames ${dname})
      endforeach()
      if(dnames)
          list(REMOVE_DUPLICATES dnames)
          foreach(dir ${dnames})
              if(NOT EXISTS "${dir}/__init__.py")
                  file(WRITE "${dir}/__init__.py" "#autogenerated by add_python_rules")
              endif()
          endforeach()
      endif()
      if(NOT EXISTS "${pythondir}/__init__.py")
          file(WRITE "${pythondir}/__init__.py" "#autogenerated by add_python_rules
import os
__path__.append(os.path.dirname(os.path.realpath(__file__)).rsplit('/src/${target}',1)[0]+'/cfipython/${target}')
")
      endif()
  endif()
  unset(pythonsrcdir)
  unset(pyfiles)
  unset(tpyfiles)
  unset(pythondir)
endmacro()

macro(add_test_properties target)
set_tests_properties(${target} PROPERTIES TIMEOUT 600 )
set_tests_properties(${target} PROPERTIES WORKING_DICRECTORY ${CMAKE_BINARY_DIR}/test/${target})
file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/test/${target})
set_tests_properties(${target}
	PROPERTIES ENVIRONMENT "LOCALRT=${CMAKE_BINARY_DIR};ROOT_INCLUDE_PATH=${CMAKE_CURRENT_SOURCE_DIR}:${CMAKE_SOURCE_DIR}:$ENV{ROOTSYS}/include:$ENV{SPACK_INCLUDE_DIRS}:$ENV{ROOT_INCLUDE_PATH};CMSSW_BASE=${CMAKE_BINARY_DIR};CMSSW_RELEASE_BASE=${CMAKE_BINARY_DIR};CMSSW_SEARCH_PATH=${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}:${CMAKE_BINARY_DIR}/test:${CMAKE_BINARY_DIR}/src;PYTHONPATH=${CMAKE_BINARY_DIR}/python:${CMAKE_BINARY_DIR}/cfipython:${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}:${CMAKE_BINARY_DIR}/test:$ENV{PYTHONPATH};PATH=${CMAKE_BINARY_DIR}/test:${CMAKE_BINARY_DIR}/bin:$ENV{PATH};LD_LIBRARY_PATH=${CMAKE_CURRENT_BINARY_DIR}:${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}:${CMAKE_BINARY_DIR}/test:$ENV{SPACK_LINK_DIRS}:$ENV{LD_LIBRARY_PATH};LOCAL_TMP_DIR=${CMAKE_BINARY_DIR}/test/${target};LOCAL_TEST_DIR=${CMAKE_CURRENT_SOURCE_DIR}")
endmacro()

add_subdirectory(Alignment)
add_subdirectory(AnalysisDataFormats)
add_subdirectory(CUDADataFormats)
add_subdirectory(CalibCalorimetry)
add_subdirectory(CalibFormats)
add_subdirectory(CalibTracker)
add_subdirectory(Calibration)
add_subdirectory(CondCore)
add_subdirectory(CondFormats)
add_subdirectory(CondTools)
add_subdirectory(DPGAnalysis)
add_subdirectory(DataFormats)
add_subdirectory(EgammaAnalysis)
add_subdirectory(FWCore)
add_subdirectory(FastSimDataFormats)
add_subdirectory(Fireworks)
add_subdirectory(Geometry)
add_subdirectory(HeterogeneousCore)
add_subdirectory(IOPool)
add_subdirectory(IORawData)
add_subdirectory(JetMETCorrections)
add_subdirectory(L1Trigger)
add_subdirectory(MuonAnalysis)
add_subdirectory(OnlineDB)
add_subdirectory(PhysicsTools)
add_subdirectory(RecoEgamma)
add_subdirectory(RecoLuminosity)
add_subdirectory(RecoMuon)
add_subdirectory(RecoParticleFlow)
add_subdirectory(RecoPixelVertexing)
add_subdirectory(RecoTracker)
add_subdirectory(SimDataFormats)
add_subdirectory(SimGeneral)
add_subdirectory(SimMuon)
add_subdirectory(SimTracker)
add_subdirectory(TBDataFormats)
add_subdirectory(TrackingTools)
add_subdirectory(Validation)

install(DIRECTORY ${CMAKE_BINARY_DIR}/src DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY ${CMAKE_BINARY_DIR}/python DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR} DESTINATION ${CMAKE_INSTALL_PREFIX})
