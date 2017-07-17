FIND_PACKAGE(R REQUIRED)

UNSET(TARGET_SWIGFLAGS)

include(ShogunInterfaces)
SET(SWIG_CXX_COMPILER_FLAGS "${SWIG_CXX_COMPILER_FLAGS} ${R_CXX_FLAGS}")
IF(APPLE)
	SET(SWIG_CXX_COMPILER_FLAGS "${SWIG_CXX_COMPILER_FLAGS} -DDO_NOT_USE_CXX_HEADERS")
ENDIF()

GENERATE_INTERFACE_TARGET(r ${CMAKE_CURRENT_SOURCE_DIR} ${R_LIBRARIES})
set_target_properties(interface_r PROPERTIES PREFIX "")

ADD_CUSTOM_COMMAND(TARGET interface_r
	POST_BUILD
	COMMAND echo 'wd="${CMAKE_CURRENT_BINARY_DIR}" \; setwd( wd ) \; f="shogun.R" \; fdata="shogun.RData" \; source( f ) \; save( list=ls( all=TRUE ) , file=fdata , compress=TRUE ) \; q( save="no" ) \;' | ${R_EXECUTABLE} --silent --no-save
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
	COMMENT "Generating shogun.RData"
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/shogun.so
	DESTINATION ${R_COMPONENT_LIB_PATH}/shogun/libs
	COMPONENT r)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/shogun.RData
	DESTINATION ${R_COMPONENT_LIB_PATH}/shogun/R
	COMPONENT r)

install(CODE "FILE(MAKE_DIRECTORY \$ENV{DESTDIR}/${R_COMPONENT_LIB_PATH}/shogun/Meta)")

install(CODE "EXECUTE_PROCESS(COMMAND ${CMAKE_SOURCE_DIR}/src/.r-install.sh \$ENV{DESTDIR}/${R_COMPONENT_LIB_PATH} shogun so saveRDS WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})")