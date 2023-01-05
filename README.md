# Example of monitor mode on nRF 52840 DK with only gcc and gdb (no Ozone or SES needed)

This is a port of BLE monitor mode example of nRF to pure make+gcc build, which requires
only GDB to debug. This means that Segger Embedded Studio or Ozone is not required to debug.
Makes possible to debug with other debuggers, such as CLion.

`CMakeLists.txt` is only just for CLion to resolve symbols, see build below.

## Build

Copy this whole directory into `nRF5_SDK_17.x.x_xxxx/examples/ble_peripheral/`

    make -C pca10056/s140/armgcc/ MMD=1
    make flash
    make flash_softdevice

To build with monitor mode, use `make MMD=1`, otherwise it's built without it.

By default, the build is unoptimized with `-O0`. You may want to change that
to `-Os` or `-Og` (look at the Makefile).

## Using in GDB or anything that is on top of GDB server

This requires JLink Plus/JTrace, the onboard JLink on the DK board won't suffice.

Run `JLinkGDBServerExe`, then connect to it from gdb via `target extended-remote :2331`
(or equivalent setting e.g. in CLion as "Remote debug")

To activate monitor mode on the JLink use this in GDB prompt:

        mon exec SetMonModeDebug = 1
        mon exec SetMonModeVTableAddr=0x27000
        monitor reset

The `0x27000` address is VTOR address, which can be found from linker script or
looking into GDB at the VTOR address:

        >>> p __isr_vector
        $1 = {<text variable, no debug info>} 0x27000 <__isr_vector>

Now you can set breakpoints and step through code without softdevice crashing on
breakpoint.

Seems that if you want to look at RTT, RTT viewer needs to be run before GDB server,
then use in GDB.

## How it works internally

If you look at the start of `Makefile`, there are few files added, that override
`DebugMon_Handler` interrupt. You also need at the beginning of main set the priority
of `DebugMonitor_IRQn` like

    NVIC_SetPriority(DebugMonitor_IRQn, _PRIO_SD_LOW);

These things combined with the monitor commands from GDB make JLink/JTrace send periodic
interrupts to keep the softdevice alive even when rest of CPU is stopped on breakpoint.

## References

* https://wiki.segger.com/nRF52_Series_Devices#Monitor_Mode_Debugging_on_Nordic_nRF52
* https://www.segger.com/products/debug-probes/j-link/technology/monitor-mode-debugging/
* https://devzone.nordicsemi.com/nordic/nordic-blog/b/blog/posts/segger-embedded-studio-part-2-monitor-mode-debuggi
* https://devzone.nordicsemi.com/nordic/nordic-blog/b/blog/posts/monitor-mode-debugging-with-j-link-and-gdbeclipse
