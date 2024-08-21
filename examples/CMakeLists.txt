SET(scr_files 00_hello-engine 01_hello-engine-raw 02_spheres 03_spinning-cube 04_spinning-primitives-tex 05_obj-viewer 06_custom-mesh-layout 07_matcap 08_gui 09_picking 10_skybox-example 11_sprite-example 12_render-to-texture  13_benchmark64k 14_spinning-primitives-openvr 15_cloth_simulation  16_shadows 17_deferred-shading)

# Create custom build targets
FOREACH(scr_file ${scr_files})
    add_executable(SRE-Example-${scr_file} ${scr_file}.cpp)
    target_link_libraries(SRE-Example-${scr_file} SRE ${SRE_LIBRARIES})
ENDFOREACH()

file(COPY ../examples_data DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
IF (WIN32)
    file(COPY ${SRE_LIBRARIES_SHARED} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
ENDIF(WIN32)
