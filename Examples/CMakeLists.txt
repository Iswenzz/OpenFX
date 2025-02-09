set(OFX_SUPPORT_HEADER_DIR "${CMAKE_CURRENT_SOURCE_DIR}/include")
file(GLOB_RECURSE OFX_SUPPORT_HEADER_FILES "${OFX_SUPPORT_HEADER_DIR}/*.h")

set(PLUGINS
	Basic
	Custom
	DepthConverter
	Invert
	OpenGL
	Rectangle
	Test
	)

foreach(PLUGIN IN LISTS PLUGINS)
	file(GLOB_RECURSE PLUGIN_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/${PLUGIN}/*.cpp")

        set(TGT example-${PLUGIN})
	add_ofx_plugin(${TGT} ${PLUGIN})
	target_sources(${TGT} PUBLIC ${PLUGIN_SOURCES})
	target_link_libraries(${TGT})
	target_include_directories(${TGT} PUBLIC ${OFX_HEADER_DIR})
endforeach()

target_link_libraries(example-OpenGL PRIVATE OpenGL::GL)
target_link_libraries(example-Custom PRIVATE OpenGL::GL)
