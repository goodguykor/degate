find_package(PkgConfig)
include_directories (.)

#
# Check libraries.
#

pkg_check_modules(LIBXML++ REQUIRED libxml++-2.6)
include_directories(${LIBXML++_INCLUDE_DIRS})
link_directories(${LIBXML++_LIBRARY_DIRS}) 
set(LIBS ${LIBS} ${LIBXML++_LIBRARIES})


find_package(TIFF REQUIRED)
include_directories(${TIFF_INCLUDE_DIR})
set(LIBS ${LIBS} ${TIFF_LIBRARIES})

find_package(JPEG REQUIRED)
include_directories(${JPEG_INCLUDE_DIR})
set(LIBS ${LIBS} ${JPEG_LIBRARIES})

#set(Boost_DEBUG ON)
set(Boost_USE_STATIC_LIBS   OFF)
find_package(Boost REQUIRED COMPONENTS filesystem system thread)
if(Boost_FOUND)
	include_directories(${Boost_INCLUDE_DIRS})
	set(LIBS ${LIBS} ${Boost_LIBRARIES})
endif()

find_package(XMLRPC COMPONENTS c++2 client)
if(XMLRPC_FOUND)
	include_directories(${XMLRPC_INCLUDE_DIRS})
	set(LIBS ${LIBS} ${XMLRPC_LIBRARIES})
endif()


pkg_check_modules(LIBZIP REQUIRED libzip)
include_directories(${LIBZIP_INCLUDE_DIRS})
link_directories(${LIBZIP_LIBRARY_DIRS}) 
set(LIBS ${LIBS} ${LIBZIP_LIBRARIES})


#
# Check endianess
#
INCLUDE(${CMAKE_ROOT}/Modules/TestBigEndian.cmake)
TEST_BIG_ENDIAN(WORDS_BIGENDIAN)

#
# The source files
#
set(SOURCES

	#
	# genric stuff
	#

	debug.cc
	FileSystem.cc
	RegularGrid.cc
	IrregularGrid.cc
	Configuration.cc
	DegateHelper.cc

	#
	# shape related
	#

	BoundingBox.cc
	Circle.cc
	Point.cc
	Line.cc
	Rectangle.cc
	TangencyCheck.cc

	#
	# logic model related
	#

	DeepCopyable.cc
	LogicModel.cc
	LogicModelHelper.cc
	Net.cc
	Layer.cc
	Project.cc
	Wire.cc
	Via.cc
	EMarker.cc
	Annotation.cc
	SubProjectAnnotation.cc
	LogicModelObjectBase.cc
	ConnectedLogicModelObject.cc
	PlacedLogicModelObject.cc
	RemoteObject.cc
	GatePort.cc
	Gate.cc
	GateTemplate.cc
	GateLibrary.cc
	Module.cc
	CodeTemplateGenerator.cc
	VHDLCodeTemplateGenerator.cc
	VHDLTBCodeTemplateGenerator.cc
	VerilogCodeTemplateGenerator.cc
	VerilogTBCodeTemplateGenerator.cc
	VerilogModuleGenerator.cc
	XmlRpc.cc
	ObjectSet.cc
	HlObjectSet.cc
	AutoNameGates.cc

	#
	# importer / exporter
	#

	Importer.cc
	XMLImporter.cc
	ProjectImporter.cc
	LogicModelImporter.cc
	GateLibraryImporter.cc
	GateLibraryExporter.cc
	LogicModelExporter.cc
	ProjectExporter.cc
	DOTExporter.cc
	DOTAttributes.cc
	LogicModelDOTExporter.cc
	ProjectArchiver.cc
	RCVBlacklistExporter.cc
	RCVBlacklistImporter.cc

	#
	# Images
	#

	FilterKernel.cc
	EdgeDetection.cc
	CannyEdgeDetection.cc
	ZeroCrossingEdgeDetection.cc
	BinaryLineDetection.cc
	WireMatching.cc
	ViaMatching.cc
	TemplateMatching.cc
	ExternalMatching.cc
	Otsu.cc

	#
	# Design Rule Checks
	#
	ERCOpenPorts.cc
	ERCNet.cc
	RCVContainer.cc
)

configure_file(Configuration.h.in ${PROJECT_SOURCE_DIR}/lib/Configuration.h)

add_library(degate SHARED ${SOURCES} )
target_link_libraries(degate ${LIBS})

#
# Documentation
#

find_package(Doxygen QUIET)
IF (DOXYGEN_FOUND)
   add_custom_target(libdoc 
     COMMAND ${DOXYGEN_EXECUTABLE} "${PROJECT_SOURCE_DIR}/doc/Doxyfile.libdegate"
     SOURCES ${SOURCES})
else()
   message("Warning: Doxygen not found. Ignoring. You should "
   	"install the doxygen package to generate the documentation for libdegate.")
ENDIF(DOXYGEN_FOUND)


if(WIN32)
    set(_library_dir bin)   # .dll are in PATH, like executables
else()
    set(_library_dir lib)
endif()

set(LIB_INSTALL_DIR "${_library_dir}${LIB_SUFFIX}")

set(_INSTALL_DESTINATIONS
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION ${LIB_INSTALL_DIR}
    ARCHIVE DESTINATION "lib${LIB_SUFFIX}"
)

install(TARGETS degate ${_INSTALL_DESTINATIONS})

