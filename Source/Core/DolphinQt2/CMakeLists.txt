include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_definitions(-DQT_USE_QSTRINGBUILDER -DQT_NO_CAST_FROM_ASCII -DQT_NO_CAST_TO_ASCII)
set(CMAKE_AUTOMOC ON)

set(SRCS
	AboutDialog.cpp
	Host.cpp
	Main.cpp
	MainWindow.cpp
	MenuBar.cpp
	RenderWidget.cpp
	Resources.cpp
	Settings.cpp
	ToolBar.cpp
	Config/PathDialog.cpp
	Config/SettingsWindow.cpp
	GameList/GameFile.cpp
	GameList/GameList.cpp
	GameList/GameListModel.cpp
	GameList/GameTracker.cpp
	GameList/ListProxyModel.cpp
	GameList/TableDelegate.cpp
	)

list(APPEND LIBS core uicommon)

set(DOLPHINQT2_BINARY dolphin-emu-qt2)

add_executable(${DOLPHINQT2_BINARY} ${SRCS} ${UI_HEADERS})
target_link_libraries(${DOLPHINQT2_BINARY} ${LIBS} Qt5::Widgets)

if(APPLE)
	# Note: This is copied from DolphinQt, based on the DolphinWX version.

	include(BundleUtilities)
	set(BUNDLE_PATH ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/${DOLPHINQT2_BINARY}.app)

	# Ask for an application bundle.
	set_target_properties(${DOLPHINQT2_BINARY} PROPERTIES
		MACOSX_BUNDLE true
		MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in
		)

	# get rid of any old copies
	file (REMOVE_RECURSE ${BUNDLE_PATH}/Contents/Resources/Sys)
	if(NOT SKIP_POSTPROCESS_BUNDLE)
		# Fix up the bundle after it is finished.
		# There does not seem to be an easy way to run CMake commands post-build,
		# so we invoke CMake again on a generated script.
		file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/postprocess_bundle.cmake "
			include(BundleUtilities)
			message(\"Fixing up application bundle: ${BUNDLE_PATH}\")
			message(\"(Note: This is only necessary to produce a redistributable binary.\")
			message(\"To skip, pass -DSKIP_POSTPROCESS_BUNDLE=1 to cmake.)\")
			set(BU_CHMOD_BUNDLE_ITEMS ON)
			execute_process(COMMAND ${CMAKE_SOURCE_DIR}/Tools/deploy-mac.py -p platforms/libqcocoa.dylib \"${BUNDLE_PATH}\")
			file(INSTALL ${CMAKE_SOURCE_DIR}/Data/Sys
				DESTINATION ${BUNDLE_PATH}/Contents/Resources
				)
			")
		add_custom_command(TARGET ${DOLPHINQT2_BINARY} POST_BUILD
			COMMAND ${CMAKE_COMMAND} -P postprocess_bundle.cmake
			)
	else()
		add_custom_command(OUTPUT ${BUNDLE_PATH}/Contents/Resources/Sys
			COMMAND ln -nfs ${CMAKE_SOURCE_DIR}/Data/Sys ${BUNDLE_PATH}/Contents/Resources/Sys
			VERBATIM
			)
		add_custom_target(CopyDataIntoBundleQt ALL
			DEPENDS ${BUNDLE_PATH}/Contents/Resources/Sys
			)
	endif()
else()
	install(TARGETS ${DOLPHINQT2_BINARY} RUNTIME DESTINATION ${bindir})
endif()
