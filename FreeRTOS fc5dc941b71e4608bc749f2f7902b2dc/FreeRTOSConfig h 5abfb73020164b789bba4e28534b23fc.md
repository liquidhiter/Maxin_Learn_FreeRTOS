# FreeRTOSConfig.h

- kernel configuration
    - application specific
- application can’t include the `FreeRTOSConfig.h` directly, but `FreeRTOS.h` instead, where `FreeRTOSConfig.h` is included

```c
#ifndef FREERTOS_CONFIG_H
#define FREERTOS_CONFIG_H

/*-----------------------------------------------------------
 * Application specific definitions.
 *
 * These definitions should be adjusted for your particular hardware and
 * application requirements.
 *
 * THESE PARAMETERS ARE DESCRIBED WITHIN THE 'CONFIGURATION' SECTION OF THE
 * FreeRTOS API DOCUMENTATION AVAILABLE ON THE FreeRTOS.org WEB SITE AND IN THE
 * FreeRTOS REFERENCE MANUAL.
 *----------------------------------------------------------*/

#define configUSE_PREEMPTION					1
#define configUSE_PORT_OPTIMISED_TASK_SELECTION	1
#define configMAX_PRIORITIES					5
#define configUSE_IDLE_HOOK						0
#define configUSE_TICK_HOOK						0
#define configTICK_RATE_HZ						( 100 ) /* This is a simulated environment and therefore not real-time. */
#define configMINIMAL_STACK_SIZE				( ( unsigned short ) 50 ) /* In this simulated case, the stack only has to hold one small structure as the real stack is part of the win32 thread. */
#define configTOTAL_HEAP_SIZE					( ( size_t ) ( 20 * 1024 ) )
#define configMAX_TASK_NAME_LEN					( 12 )
#define configUSE_TRACE_FACILITY				0
#define configUSE_16_BIT_TICKS					0
#define configIDLE_SHOULD_YIELD					1
#define configUSE_MUTEXES						1
#define configCHECK_FOR_STACK_OVERFLOW			0 /* Not applicable when using the Win32 simulator. */
#define configUSE_RECURSIVE_MUTEXES				1
#define configQUEUE_REGISTRY_SIZE				10
#define configUSE_MALLOC_FAILED_HOOK			1
#define configUSE_APPLICATION_TASK_TAG			0
#define configUSE_COUNTING_SEMAPHORES			1
#define configUSE_ALTERNATIVE_API				0
#define configUSE_QUEUE_SETS					1

/* Software timer related configuration options. */
#define configUSE_TIMERS						0
#define configTIMER_TASK_PRIORITY				( configMAX_PRIORITIES - 1 )
#define configTIMER_QUEUE_LENGTH				20
#define configTIMER_TASK_STACK_DEPTH			( configMINIMAL_STACK_SIZE * 2 )

/* Run time stats gathering configuration options. */
#define configGENERATE_RUN_TIME_STATS			0

/* Co-routine related configuration options. */
#define configUSE_CO_ROUTINES 					0
#define configMAX_CO_ROUTINE_PRIORITIES 		2

/* This demo does not make use of one or more example stats formatting
functions, which format the raw data provided by the uxTaskGetSystemState()
function in to human readable ASCII form. */
#define configUSE_STATS_FORMATTING_FUNCTIONS	0

/* Set the following definitions to 1 to include the API function, or zero
to exclude the API function.  In most cases the linker will remove unused
functions anyway. */
#define INCLUDE_vTaskPrioritySet				1
#define INCLUDE_uxTaskPriorityGet				1
#define INCLUDE_vTaskDelete						1
#define INCLUDE_vTaskSuspend					1
#define INCLUDE_vTaskDelayUntil					1
#define INCLUDE_vTaskDelay						1
#define INCLUDE_uxTaskGetStackHighWaterMark		1
#define INCLUDE_xTaskGetSchedulerState			1
#define INCLUDE_xTimerGetTimerDaemonTaskHandle	1
#define INCLUDE_xTaskGetIdleTaskHandle			1
#define INCLUDE_pcTaskGetTaskName				1
#define INCLUDE_eTaskGetState					1
#define INCLUDE_xSemaphoreGetMutexHolder		1
#define INCLUDE_xTimerPendFunctionCall			1

/* It is a good idea to define configASSERT() while developing.  configASSERT()
uses the same semantics as the standard C assert() macro. */
extern void vAssertCalled( uint32_t ulLine, const char * const pcFileName );
#define configASSERT( x ) if( ( x ) == 0 ) vAssertCalled( __LINE__, __FILE__ )

#endif /* FREERTOS_CONFIG_H */

```

- implementation of `vAssertCalled`

```c
void vAssertCalled( uint32_t ulLine, const char * const pcFile )
{
/* The following two variables are just to ensure the parameters are not
optimised away and therefore unavailable when viewed in the debugger. */
volatile uint32_t ulLineNumber = ulLine, ulSetNonZeroInDebuggerToReturn = 0;
volatile const char * const pcFileName = pcFile;

	taskENTER_CRITICAL();
	{
		while( ulSetNonZeroInDebuggerToReturn == 0 )
		{
			/* If you want to set out of this function in the debugger to see
			the	assert() location then set ulSetNonZeroInDebuggerToReturn to a
			non-zero value. */
		}
	}
	taskEXIT_CRITICAL();

	/* Remove the potential for compiler warnings issued because the variables
	are set but not subsequently referenced. */
	( void ) pcFileName;
	( void ) ulLineNumber;
}
/*-----------------------------------------------------------*/
```

- tricks
    - define two new `volatile` variables taking the values of the given arguments to ensure they are not optimised by the compiler
        - does it depend on the compiler optimization level ?
        - it is because the given arguments are not consumed, and therefore, the compiler optimizes them out ?
- disassembly code

```nasm
vAssertCalledSafe:
        push    {r7, lr}
        sub     sp, sp, #24
        add     r7, sp, #0
        str     r0, [r7, #4]
        str     r1, [r7]
        ldr     r3, [r7, #4]
        str     r3, [r7, #16]
        movs    r3, #0
        str     r3, [r7, #12]
        ldr     r3, [r7]
        str     r3, [r7, #20]
        bl      taskENTER_CRITICAL
        nop
.L4:
        ldr     r3, [r7, #12]
        cmp     r3, #0
        beq     .L4
        bl      taskEXIT_CRITICAL
        ldr     r3, [r7, #16]
        nop
        adds    r7, r7, #24
        mov     sp, r7
        pop     {r7, pc}
```

- without using `volatile`
- example code

```c
void vAssertCalled( uint32_t ulLine, const char * const pcFile )
{
    {
        while (1)
        {
            /* Do nothing */
        }
    }

    /* Remove the potential for compiler warnings issued because the variables are
    set but not subsequently referenced */
    ( void ) ulLine;
    ( void ) pcFile;
}
```

```nasm
vAssertCalled:
        push    {r7}
        sub     sp, sp, #12
        add     r7, sp, #0
        str     r0, [r7, #4]
        str     r1, [r7]
.L2:
        b       .L2
```

### experiment on CC2340R5

```c
/* Testing the impact of optimization on unsed function arguments */
void vAssertCalled( uint32_t ulLine, const char * const pcFile )
{
    ( void )ulLine;
    ( void )pcFile;
}

/*
 *  ======== mainThread ========
 */
void *mainThread(void *arg0)
{

#if 1
    vAssertCalled( 32U, (void *)0);
#endif

    /* 1 second delay */
    uint32_t time = 1;

    /* Call driver init functions */
    GPIO_init();
    // I2C_init();
    // SPI_init();
    // Watchdog_init();

    /* Configure the LED pin */
    GPIO_setConfig(CONFIG_GPIO_LED_0, GPIO_CFG_OUT_STD | GPIO_CFG_OUT_LOW);

    /* Turn on user LED */
    GPIO_write(CONFIG_GPIO_LED_0, CONFIG_GPIO_LED_ON);

    while (1)
    {
        sleep(time);
        GPIO_toggle(CONFIG_GPIO_LED_0);
    }
}

```

- observation
    - optimization level `-g`

![Untitled](FreeRTOSConfig%20h%205abfb73020164b789bba4e28534b23fc/Untitled.png)

1. argument `ulLine` is passed into the register `r0`
    1. storing it into the stack, address is `SP + 4`
2. `pFile` seems to be optimized out even the dummy variable of `pcFileName`
    1. `pcFileName` is defined to be `volatile`, the compiler doesn’t guarantee to retain a `volatile` restricted variable ?
        1. compiler can eliminate the un-used local `volatile` variable from a function (reference: [https://stackoverflow.com/questions/4933314/unused-volatile-variable](https://stackoverflow.com/questions/4933314/unused-volatile-variable))
        2. 
3. compiler directly assigns value of `0` (`NULL`) which is the value of `pcFile` , and push it into the stack, at the address `SP`
    1. orders of pushing the function arguments depend on the calling convention, here it seems to be from the left to the right
    2. what will happen if the value of `pcFile` is non-null?
        1. would expect to see it is not optimized out
            1. actually, it is still optimized out even with a non-zero value, e.x. `const char * textLiteral = "FREERTOS";`
            - [ ]  how does it happen ?