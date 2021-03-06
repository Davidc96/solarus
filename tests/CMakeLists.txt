# List of maps of the testing quest that are unit tests to be run.
set(lua_test_maps
  "all_entities"
  "basic_test"
  "dynamic_tile_tests"
  "jumper_tests"
  "surface_tests"
  "teletransportation_tests/main"
  "bugs/486_diagonal_dynamic_tiles"
  "bugs/496_stream_speed_0"
  "bugs/526_get_entities_same_region"
  "bugs/633_straight_movement_speed_zero"
  "bugs/656_map_set_world"
  "bugs/664_chest_get_set_treasure"
  "bugs/667_hero_save_solid_ground_callback"
  "bugs/669_entity_get_sprite"
  "bugs/670_map_get_entities_no_hero"
  "bugs/671_collision_test_not_moving"
  "bugs/686_crash_door_item"
  "bugs/699_crash_exit_surface_moving"
  "bugs/712_npc_set_traversable"
  "bugs/716_custom_entity_set_can_traverse_door"
  "bugs/729_switch_is_walkable"
  "bugs/738_custom_entity_on_ground_below_changed"
  "bugs/748_entity_overlaps_collision_modes"
  "bugs/754_entity_get_max_bounding_box"
  "bugs/762_camera_entity"
  "bugs/769_stuck_dynamic_tile_covered"
  "bugs/779_get_entities_random_order"
  "bugs/781_assert_teletransporter_same_map"
  "bugs/794_custom_entity_set_can_traverse_ground"
  "bugs/795_recreate_entity_same_name"
  "bugs/796_map_get_entities_by_type"
  "bugs/805_hero_set_invincible"
  "bugs/807_cannot_traverse_own_ground"
  "bugs/808_get_entities_in_rectangle_duplicates"
  "bugs/809_entity_bring_sprite_to_front_back"
  "bugs/810_hero_disappears_after_scrolling/1"
  "bugs/817_entity_set_enabled_overlapping_hero"
  "bugs/818_sprite_get_num_frames"
  "bugs/819_destination_starting_location_mode/1"
  "bugs/821_hero_start_attack"
  "bugs/823_sprite_get_size_origin"
  "bugs/827_ground_detection_after_unfreeze"
  "bugs/830_entity_get_ground_position"
  "bugs/844_camera_on_state_changed"
  "bugs/845_game_set_suspended"
  "bugs/851_entity_get_sprites"
  "bugs/855_hero_get_sprite_wrong"
  "bugs/859_crash_game_start"
  "bugs/860_crash_remove_sprite"
  "bugs/861_sprite_set_animation_callback"
  "bugs/867_crash_crystal"
  "bugs/875_freeze_load_map_tiles_outside"
  "bugs/877_get_facing_entity"
  "bugs/880_custom_entity_set_origin"
  "bugs/883_collision_test_not_moving_sprite"
  "bugs/884_tileset_sprite_file_optional"
  "bugs/889_crash_use_non_saved_item"
  "bugs/937_custom_entity_missing_direction"
  "bugs/940_crash_bomb_on_stream"
  "bugs/945_flying_enemies_fall_in_hole"
  "bugs/946_reused_movement_callback"
  "bugs/954_entity_name_nil_after_removed"
  "bugs/1076_treasure_dialog_optional"
  "bugs/1094_entity_properties"
)

# Build the Solarus testing library.
file(
  GLOB
  testing_source_files
  src/test_tools/TestEnvironment.cpp
  include/test_tools/TestEnvironment.h
  include/test_tools/TestEnvironment.inl
)

include_directories(
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

add_library(solarus_testing
  SHARED
  ${testing_source_files}
)

target_link_libraries(solarus_testing
  solarus
  "${SDL2_LIBRARY}"
  "${SDL2_IMAGE_LIBRARY}"
  "${SDL2_TTF_LIBRARY}"
  "${OPENAL_LIBRARY}"
  "${LUA_LIBRARY}"
  "${DL_LIBRARY}"
  "${PHYSFS_LIBRARY}"
  "${VORBISFILE_LIBRARY}"
  "${OGG_LIBRARY}"
  "${MODPLUG_LIBRARY}"
)

# Build individual tests.

# Source files of the 'src/tests' directory that are a test with a main() function.
set(
  tests_main_files
  src/tests/Initialization.cpp
  src/tests/MapData.cpp
  src/tests/LanguageData.cpp
  src/tests/PathFinding.cpp
  src/tests/PathMovement.cpp
  src/tests/PixelMovement.cpp
  src/tests/Quadtree.cpp
  src/tests/SpriteData.cpp
  src/tests/TilesetData.cpp
  src/tests/RunLuaTest.cpp
)

foreach(test_main_file ${tests_main_files})

  get_filename_component(test_bin_file ${test_main_file} NAME_WE)
  add_executable(${test_bin_file} ${test_main_file})
  
  if (WIN32)
    set(SDL2MAIN_LINK "${SDL2MAIN_LIBRARY}")
  else()
    set(SDL2MAIN_LINK "")
  endif()

  target_link_libraries(${test_bin_file}
    solarus
    solarus_testing
    "${SDL2_LIBRARY}"
    "${SDL2MAIN_LINK}"
    "${SDL2_IMAGE_LIBRARY}"
    "${SDL2_TTF_LIBRARY}"
    "${OPENAL_LIBRARY}"
    "${LUA_LIBRARY}"
    "${DL_LIBRARY}"
    "${PHYSFS_LIBRARY}"
    "${VORBISFILE_LIBRARY}"
    "${OGG_LIBRARY}"
    "${MODPLUG_LIBRARY}"
  )
  set_target_properties(${test_bin_file}
    PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/bin"
  )

  if (${test_main_file} MATCHES "src/tests/RunLuaTest.cpp")
    # Lua test runner: add an individual test for each map.
    foreach(map_id ${lua_test_maps})
      add_test("lua/${map_id}" "bin/${test_bin_file}" -no-audio -no-video -turbo=yes -map=${map_id} "${CMAKE_CURRENT_SOURCE_DIR}/testing_quest")
    endforeach()
  else()
    # Normal C++ test.
    get_filename_component(test_name "${test_main_file}" NAME_WE)
    string(TOLOWER "${test_name}" test_name)
    add_test("${test_name}" "bin/${test_bin_file}" -no-audio -no-video -turbo=yes "${CMAKE_CURRENT_SOURCE_DIR}/testing_quest")
  endif()

endforeach()

