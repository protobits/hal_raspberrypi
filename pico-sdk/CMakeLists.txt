zephyr_library()

set(RP2_COMMON_DIR ${CMAKE_CURRENT_LIST_DIR}/src/rp2_common)
set(RP2040_DIR ${CMAKE_CURRENT_LIST_DIR}/src/rp2040)
set(COMMON_DIR ${CMAKE_CURRENT_LIST_DIR}/src/common)

set(BOOT_STAGE_DIR ${RP2_COMMON_DIR}/boot_stage2)
set(PICO_SDK_PATH ${CMAKE_CURRENT_LIST_DIR})

add_executable(boot_stage2 ${BOOT_STAGE_DIR}/${CONFIG_RP2_FLASH_BOOT_FILE})

target_include_directories(boot_stage2 PUBLIC
	${CMAKE_CURRENT_LIST_DIR}
	${BOOT_STAGE_DIR}/asminclude
	${RP2_COMMON_DIR}/pico_platform/include
	${RP2040_DIR}/hardware_regs/include
	${COMMON_DIR}/pico_base/include
	)

target_link_options(boot_stage2 PRIVATE "-nostartfiles")
target_link_options(boot_stage2 PRIVATE "--specs=nosys.specs")
target_link_options(boot_stage2 PRIVATE "LINKER:--script=${BOOT_STAGE_DIR}/boot_stage2.ld")

add_custom_target(bootloader_bin ALL DEPENDS boot_stage2.bin)
add_custom_command(OUTPUT boot_stage2.bin DEPENDS boot_stage2 COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:boot_stage2> boot_stage2.bin)

add_custom_target(boot_stage2_asm ALL DEPENDS boot_stage2.S)

add_custom_command(OUTPUT boot_stage2.S DEPENDS boot_stage2.bin COMMAND ${Python3_EXECUTABLE} ${BOOT_STAGE_DIR}/pad_checksum -s 0xffffffff boot_stage2.bin boot_stage2.S)

zephyr_library_sources(boot_stage2.S
	${RP2_COMMON_DIR}/hardware_clocks/clocks.c
	${RP2_COMMON_DIR}/hardware_pll/pll.c
	${RP2_COMMON_DIR}/hardware_xosc/xosc.c
	${RP2_COMMON_DIR}/hardware_watchdog/watchdog.c
	${RP2_COMMON_DIR}/hardware_gpio/gpio.c
	${RP2_COMMON_DIR}/hardware_pio/pio.c
	${RP2_COMMON_DIR}/hardware_dma/dma.c
	${RP2_COMMON_DIR}/hardware_claim/claim.c
	${RP2_COMMON_DIR}/hardware_sync/sync.c
	${RP2_COMMON_DIR}/hardware_uart/uart.c
	${RP2_COMMON_DIR}/pico_platform/platform.c
	${RP2_COMMON_DIR}/pico_bootrom/bootrom.c
)

zephyr_include_directories(
	${RP2_COMMON_DIR}/hardware_base/include
	${RP2_COMMON_DIR}/hardware_clocks/include
	${RP2_COMMON_DIR}/hardware_watchdog/include
	${RP2_COMMON_DIR}/hardware_xosc/include
	${RP2_COMMON_DIR}/hardware_pll/include
	${RP2_COMMON_DIR}/hardware_vreg/include
	${RP2_COMMON_DIR}/hardware_irq/include
	${RP2_COMMON_DIR}/hardware_gpio/include
	${RP2_COMMON_DIR}/hardware_claim/include
	${RP2_COMMON_DIR}/hardware_sync/include
	${RP2_COMMON_DIR}/hardware_pio/include
	${RP2_COMMON_DIR}/hardware_dma/include
	${RP2_COMMON_DIR}/hardware_sync/include
	${RP2_COMMON_DIR}/hardware_uart/include
	${RP2_COMMON_DIR}/hardware_timer/include
	${RP2_COMMON_DIR}/hardware_resets/include
	${RP2040_DIR}/hardware_regs/include
	${RP2040_DIR}/hardware_structs/include
	${COMMON_DIR}/pico_base/include
	${COMMON_DIR}/pico_binary_info/include
	${RP2_COMMON_DIR}/pico_platform/include
	${RP2_COMMON_DIR}/pico_bootrom/include
	${CMAKE_CURRENT_LIST_DIR}
)

list(APPEND CMAKE_MODULE_PATH ${ZEPHYR_RASPBERRYPI_CMAKE_DIR}/pico-sdk/tools)
find_package(Pioasm REQUIRED)