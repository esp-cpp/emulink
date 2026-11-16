# emulink: the emulators' USB link, firmware side.
#
# - DevLink (include/devlink.hpp): the espp dispatcher on the USB vendor
#   interface, with the espp system / monitor / coredump services
# - RemoteControl (include/remote_control.hpp): the emulator's test control
#   (module 0x20, "emu.control"), header only: the app's main includes it
#
# The emulator's board component provides box-emu.hpp (class BoxEmu). It is
# box-emu unless the project names another before project():
#   idf_build_set_property(EMULINK_BOARD_COMPONENT tab5-emu)
idf_build_get_property(emulink_board EMULINK_BOARD_COMPONENT)
if(NOT emulink_board)
  set(emulink_board box-emu)
endif()

idf_component_register(
  INCLUDE_DIRS "include"
  SRCS "src/devlink.cpp"
  PRIV_REQUIRES esp_system
  REQUIRES app_update coredump dispatcher monitor system ${emulink_board})
# no OTA service in the emulators: an update goes through the updater app in
# ota_1 (the Tab5's app does not fit twice; the box runs its code from PSRAM,
# and writing the flash from it corrupts its PSRAM heap)
target_compile_definitions(${COMPONENT_LIB} PUBLIC DEVLINK_HAS_OTA=0)
# keep the last panic's reason across the reset (DevLink::last_panic())
target_link_options(${COMPONENT_LIB} INTERFACE "-Wl,--wrap=esp_panic_handler" "-Wl,--wrap=panic_abort")
