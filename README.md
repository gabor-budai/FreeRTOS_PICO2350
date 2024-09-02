# FreeRTOS on Pico 2350
Currently, FreeRTOS and Visual GDB do not support the RP2350. This example demonstrates how to use the RP2350 with these tools until official support is available.

## Visual Studio Code
To get started with the RP2350, the easiest option is to use the Raspberry Pi Pico extension in VS Code. Later, you'll need it for debugging with Visual GDB.

## Visual GDB
Visual GDB's current version of OpenOCD does not support the RP2350. Initially, I attempted to use the [target/rp2350.cfg](https://github.com/raspberrypi/openocd/blob/sdk-2.0.0/tcl/target/rp2350.cfg) configuration file from Raspberry's repository, but it was incompatible with the Visual GDB's OpenOCD tool. However, running example code with VS Code worked as expected.  
To resolve this issue, follow these steps:
1. Install the [Rasberry Pi Pico](https://marketplace.visualstudio.com/items?itemName=raspberry-pi.raspberry-pi-pico) extension for VS Code. Running an example with this extension will also download the necessary OpenOCD files.
2. Locate the Raspberry's OpenOCD files at `%userprofile%\.pico-sdk\openocd\{rev}`.
3. Integration with Visual GDB 
    - The Visual GDB's OpenOCD files at `%localappdata%\VisualGDB\EmbeddedDebugPackages\com.sysprogs.arm.openocd`.
    - Copy scripts to `share\openocd\scripts` and the remaining files to the `bin` folder.

It's a good idea to rename the original openocd.exe in Visual GDB before overwriting it. While I don't anticipate compatibility issues, as I've tested the updated OpenOCD with an H723 and it worked, renaming the original file helps ensure that you can revert if needed.

After completing these steps, you can create or modify a *Raspberry Pi Pico Project* in Visual GDB. In the project creation under Debug Method or in the VisualGDB Project Properties under Debug Settings, configure the following:
- *Debug using*: OpenOCD
- *JTAG/SWD programmer*: CMSIS-DAP
- *Debugged device*: target/rp2350.cfg

## FreeRTOS Port Update
I was unaware of the Raspberry Pi team's [FreeRTOS-Kernel](https://github.com/raspberrypi/FreeRTOS-Kernel) fork before I started this project. I've updated the project to use the RPI fork, so you can now use this repo as an example.

## FreeRTOS Port
The RP2350 uses two processor architectures: M0PLUS and M33. I can cover the M33 architecture. Although I'm not entirely sure, I believe the RP2040 and RP2350 ports are similar, except for one key difference: the spin wait implementation.  

The RP2350's errata indicate that SIO SPINLOCK writes may be spuriously detected. The M0PLUS core lacks atomic support, so hardware spin lock support was mandatory. However, since the M33 core supports hardware atomic operations, spin waits fall back to a software implementation. According to the datasheet, reading a (hardware) spin lock acquires the lock, but this approach is not feasible with the software implementation because it cause the code to get stuck. Instead, an API must be used to acquire the lock, with atomic instructions providing the same functionality as the hardware did. This API was introduced in pico-sdk version 2.0.0.  
  
In the port macro, the locking sections need to be updated, and the RP2040 port can be adapted for the RP2350. This modification does not affect the RP2040 port. Therefore, rather than creating a custom port, I have updated the existing RP2040 port. The official implementation might be more complex as it would also need to support the RISC-V architecture. (The portmacro can be found at src\\FreeRTOS\\portable\\ThirdParty\\GCC\\RP2040)

Truncated `portmacro.h`:  
```c
static inline void vPortRecursiveLock( uint32_t ulLockNum, spin_lock_t * pxSpinLock, BaseType_t uxAcquire )
{
    // recursion logic...

    if( uxAcquire )
    {
        //if( __builtin_expect( !*pxSpinLock, 0 ) )
        if ( __builtin_expect(!spin_try_lock_unsafe(pxSpinLock), 0 ) )
        {
            // recursion logic (if aquired, returns)
            // return;

            spin_lock_unsafe_blocking(pxSpinLock);
            //while( __builtin_expect( !*pxSpinLock, 0 ) ) {}
        }

        //__mem_fence_acquire(); // lock does this.
        // recursion logic
    }
    else
    {
        // recursion logic (release if recursion count is 0)
        spin_unlock_unsafe(pxSpinLock);
        //__mem_fence_release(); // unlock does this.
        //*pxSpinLock = 1;
        
    }
}
```

## Segger J-Link
The J-Link currently does not support the RP2350. It is possible to convert it into an OpenOCD debugger, which then allows it to be used with the RP2350. However, the official [Raspberry Pi Debug Probe](https://www.raspberrypi.com/products/debug-probe/) is one of the most affordable debuggers available. I recommend purchasing it instead of modifying a J-Link, as it offers a simpler solution. While it may be a bit slower, this is not a dealbreaker.  
Alternatively, if you already have an RP2040, you can convert it into a [debugger](https://github.com/raspberrypi/debugprobe).