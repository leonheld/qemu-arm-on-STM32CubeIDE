# Changes you need to make to run images on qemu-arm built with STM32CubeIDE

I'm talking about this qemu-arm: https://xpack.github.io/qemu-arm/

You may get this error when trying to run images build in CubeIDE:

```NVIC: Bad read offset 0xd88              
C:\Users\leon\Documents\bin\xpack-qemu-arm-2.8.0-10\bin\qemu-system-gnuarmeclipse.exe: Attempt to set CP10/11 in SCB->CPACR, but FP is not supported yet.
```

1. Change this section of the linker:

```  .data :
  {
    . = ALIGN(4);
    _sdata = .;        /* create a global symbol at data start */
    *(.data)           /* .data sections */
    *(.data*)          /* .data* sections */

    . = ALIGN(4);
    _edata = .;        /* define a global symbol at data end */
   
  } >RAM AT> FLASH
```

to 

```    .data : AT (_sidata)
    {
		_sdata = .;
		*(.data)		/* Initialized data */
		_edata = .;
    } >RAM
```

2. Disable the FPU (selecting software floating point)

Properties -> C/C++ Build -> Settings -> MCU Settings -> Floating point unit = None and below select Software Implementation (-mfloat-abi=soft). You can manually change this settings on the linker as well.

3. Comment out the FPU initilization code

```void SystemInit(void)
{
  /* FPU settings ------------------------------------------------------------*/
// #if (__FPU_PRESENT == 1) && (__FPU_USED == 1)
//   SCB->CPACR |= ((3UL << 10*2)|(3UL << 11*2));  /* set CP10 and CP11 Full Access */
```

on the file called system_stm32f4xx.c in the /src directory of your build.

4. Build clean and rebuild, now it works fine :-)
