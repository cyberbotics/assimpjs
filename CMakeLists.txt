cmake_minimum_required (VERSION 3.6)

set_property (GLOBAL PROPERTY USE_FOLDERS ON)

set (CMAKE_SUPPRESS_REGENERATION 1)
set (CMAKE_CONFIGURATION_TYPES Debug;Release;RelWithDebInfo;MinSizeRel)

add_definitions (-DUNICODE -D_UNICODE)

project (AssimpJS)

function (SetCompilerOptions module)
	target_compile_options (${module} PUBLIC "$<$<CONFIG:Debug>:-DDEBUG>")
	if (WIN32)
		set (AdditionalWarnings /w44061 /w44062 /w44265 /w44266 /w44355 /w44596 /w44800)
		target_compile_options (${module} PUBLIC /W3 ${AdditionalWarnings})
	else ()
		target_compile_options (${module} PUBLIC -std=c++11)
	endif ()
endfunction ()

# Assimp
set (BUILD_SHARED_LIBS OFF CACHE BOOL "")
set (ASSIMP_BUILD_TESTS OFF CACHE BOOL "")
set (ASSIMP_BUILD_ASSIMP_TOOLS OFF CACHE BOOL "")
set (ASSIMP_BUILD_ALL_EXPORTERS_BY_DEFAULT OFF CACHE BOOL "")
set (ASSIMP_BUILD_ASSJSON_EXPORTER ON CACHE BOOL "")
set (ASSIMP_BUILD_GLTF_EXPORTER ON CACHE BOOL "")

# undefined behaviour because of alignment issues, crashes and memory leaks
# https://github.com/assimp/assimp/pull/4044
set (ASSIMP_BUILD_M3D_IMPORTER OFF CACHE BOOL "")

# assertion failed because of a memory error
set (ASSIMP_BUILD_RAW_IMPORTER OFF CACHE BOOL "")

# removed from assimp, so doesn't need to compile at all
# https://github.com/assimp/assimp/issues/3647
set (ASSIMP_BUILD_X3D_IMPORTER OFF CACHE BOOL "")

# doesn't work because of xml parsing error
set (ASSIMP_BUILD_IRR_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_IRRMESH_IMPORTER OFF CACHE BOOL "")

# doesn't work
set (ASSIMP_BUILD_TERRAGEN_IMPORTER OFF CACHE BOOL "")

# this dramatically decreases the wasm file size
set (ASSIMP_BUILD_IFC_IMPORTER OFF CACHE BOOL "")

# not needed
set (ASSIMP_BUILD_3DS_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_3MF_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_AC_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_AMF_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_ASE_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_Assbin_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_B3D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_Blender_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_BVH_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_C4D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_COB_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_CSM_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_DXF_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_HMP_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_IFC_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_IQM_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_LWO_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_LWS_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_M3D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MD2_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MD3_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MD4_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MD5_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MDC_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MDL_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MMD_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_MS3D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_NDO_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_NFF_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_OFF_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_OGRE_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_OPENGEX_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_PLY_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_Q3BSP_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_Q3D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_RAW_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_SIB_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_SMD_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_STEP_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_STEPParser_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_TERRAGEN_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_UNREAL_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_X_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_X3D_IMPORTER OFF CACHE BOOL "")
set (ASSIMP_BUILD_XGL_IMPORTER OFF CACHE BOOL "")

add_subdirectory (assimp)

if (${EMSCRIPTEN})
	target_compile_options (assimp PUBLIC -fexceptions)
	target_compile_options (assimp PUBLIC -Wno-unused-but-set-variable)
endif ()

# AssimpJS

set (AssimpJSSourcesFolder assimpjs/src)
file (GLOB
	AssimpJSSourceFiles
	${AssimpJSSourcesFolder}/*.hpp
	${AssimpJSSourcesFolder}/*.cpp
)
source_group ("Sources" FILES ${AssimpJSSourceFiles})

if (${EMSCRIPTEN})
	add_executable (AssimpJS ${AssimpJSSourceFiles})
	target_compile_options (AssimpJS PUBLIC "$<$<CONFIG:Debug>:-gsource-map>")
	target_compile_options (AssimpJS PUBLIC -fexceptions)

	target_link_options (AssimpJS PUBLIC -sMODULARIZE=1)
	target_link_options (AssimpJS PUBLIC -sEXPORT_NAME='assimpjs')
	target_link_options (AssimpJS PUBLIC -sALLOW_MEMORY_GROWTH=1 --no-heap-copy)
	target_link_options (AssimpJS PUBLIC -sNO_DISABLE_EXCEPTION_CATCHING)
	
	# to check for memory errors
	# target_link_options (AssimpJS PUBLIC -sASSERTIONS=1 -sSAFE_HEAP=1 -sWARN_UNALIGNED=1)
	
	target_link_options (AssimpJS PUBLIC --bind)
else ()
	add_library (AssimpJS ${AssimpJSSourceFiles})
endif ()

SetCompilerOptions (AssimpJS)
target_link_libraries (AssimpJS assimp)
set_target_properties(AssimpJS PROPERTIES OUTPUT_NAME assimpjs)
set_target_properties (AssimpJS PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}")

# AssimpJSTest

if (${EMSCRIPTEN})
else ()
	set (AssimpJSExampleSourcesFolder assimpjs/example)
	file (GLOB
		AssimpJSExampleSourceFiles
		${AssimpJSExampleSourcesFolder}/*.hpp
		${AssimpJSExampleSourcesFolder}/*.cpp
	)
	source_group ("Sources" FILES ${AssimpJSExampleSourceFiles})
	add_executable (AssimpJSExample ${AssimpJSExampleSourceFiles})
	SetCompilerOptions (AssimpJSExample)
	target_include_directories (AssimpJSExample PUBLIC ${AssimpJSSourcesFolder} ${TinyCSGEmscriptenSourcesFolder})
	target_link_libraries (AssimpJSExample AssimpJS)
	set_target_properties (AssimpJSExample PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}")
endif ()
