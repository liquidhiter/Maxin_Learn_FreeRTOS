# portmacro.h

> MCU (architecture) and compiler specific
> 
- example
    - GCC
    - ARM Cortex M0p

```c
#ifndef PORTMACRO_H
    #define PORTMACRO_H

    #ifdef __cplusplus
        extern "C" {
    #endif

/*-----------------------------------------------------------
 * Port specific definitions.
 *
 * The settings in this file configure FreeRTOS correctly for the
 * given hardware and compiler.
 *
 * These settings should not be altered.
 *-----------------------------------------------------------
 */

/* Type definitions. */
    #define portCHAR          char
    #define portFLOAT         float
    #define portDOUBLE        double
    #define portLONG          long
    #define portSHORT         short
    #define portSTACK_TYPE    uint32_t
    #define portBASE_TYPE     long

    typedef portSTACK_TYPE   StackType_t;
    typedef long             BaseType_t;
    typedef unsigned long    UBaseType_t;

    #if ( configUSE_16_BIT_TICKS == 1 )
        typedef uint16_t     TickType_t;
        #define portMAX_DELAY              ( TickType_t ) 0xffff
    #else
        typedef uint32_t     TickType_t;
        #define portMAX_DELAY              ( TickType_t ) 0xffffffffUL

/* 32-bit tick type on a 32-bit architecture, so reads of the tick count do
 * not need to be guarded with a critical section. */
        #define portTICK_TYPE_IS_ATOMIC    1
    #endif
/*-----------------------------------------------------------*/

/* Architecture specifics. */
    #define portSTACK_GROWTH      ( -1 )
    #define portTICK_PERIOD_MS    ( ( TickType_t ) 1000 / configTICK_RATE_HZ )
    #define portBYTE_ALIGNMENT    8
    #define portDONT_DISCARD      __attribute__( ( used ) )
/*-----------------------------------------------------------*/

/* Scheduler utilities. */
    extern void vPortYield( void );
    #define portNVIC_INT_CTRL_REG     ( *( ( volatile uint32_t * ) 0xe000ed04 ) )
    #define portNVIC_PENDSVSET_BIT    ( 1UL << 28UL )
    #define portYIELD()                                 vPortYield()
    #define portEND_SWITCHING_ISR( xSwitchRequired )    do { if( xSwitchRequired ) portNVIC_INT_CTRL_REG = portNVIC_PENDSVSET_BIT; } while( 0 )
    #define portYIELD_FROM_ISR( x )                     portEND_SWITCHING_ISR( x )
/*-----------------------------------------------------------*/

/* Critical section management. */
    extern void vPortEnterCritical( void );
    extern void vPortExitCritical( void );
    extern uint32_t ulSetInterruptMaskFromISR( void ) __attribute__( ( naked ) );
    extern void vClearInterruptMaskFromISR( uint32_t ulMask )  __attribute__( ( naked ) );

    #define portSET_INTERRUPT_MASK_FROM_ISR()         ulSetInterruptMaskFromISR()
    #define portCLEAR_INTERRUPT_MASK_FROM_ISR( x )    vClearInterruptMaskFromISR( x )
    #define portDISABLE_INTERRUPTS()                  __asm volatile ( " cpsid i " ::: "memory" )
    #define portENABLE_INTERRUPTS()                   __asm volatile ( " cpsie i " ::: "memory" )
    #define portENTER_CRITICAL()                      vPortEnterCritical()
    #define portEXIT_CRITICAL()                       vPortExitCritical()

/*-----------------------------------------------------------*/

/* Tickless idle/low power functionality. */
    #ifndef portSUPPRESS_TICKS_AND_SLEEP
        extern void vPortSuppressTicksAndSleep( TickType_t xExpectedIdleTime );
        #define portSUPPRESS_TICKS_AND_SLEEP( xExpectedIdleTime )    vPortSuppressTicksAndSleep( xExpectedIdleTime )
    #endif
/*-----------------------------------------------------------*/

/* Task function macros as described on the FreeRTOS.org WEB site. */
    #define portTASK_FUNCTION_PROTO( vFunction, pvParameters )    void vFunction( void * pvParameters )
    #define portTASK_FUNCTION( vFunction, pvParameters )          void vFunction( void * pvParameters )

    #define portNOP()

    #define portMEMORY_BARRIER()    __asm volatile ( "" ::: "memory" )

    #ifdef __cplusplus
        }
    #endif

#endif /* PORTMACRO_H */

```

- critical sections needed to read the ticks

```c
/* 32-bit tick type on a 32-bit architecture, so reads of the tick count do
 * not need to be guarded with a critical section. */
        #define portTICK_TYPE_IS_ATOMIC    1
```

- taken from the

[FreeRTOSConfig.h](FreeRTOSConfig%20h%205abfb73020164b789bba4e28534b23fc.md)

```c
#if ( portTICK_TYPE_IS_ATOMIC == 0 )

/* Either variables of tick type cannot be read atomically, or
 * portTICK_TYPE_IS_ATOMIC was not set - map the critical sections used when
 * the tick count is returned to the standard critical section macros. */
    #define portTICK_TYPE_ENTER_CRITICAL()                      portENTER_CRITICAL()
    #define portTICK_TYPE_EXIT_CRITICAL()                       portEXIT_CRITICAL()
    #define portTICK_TYPE_SET_INTERRUPT_MASK_FROM_ISR()         portSET_INTERRUPT_MASK_FROM_ISR()
    #define portTICK_TYPE_CLEAR_INTERRUPT_MASK_FROM_ISR( x )    portCLEAR_INTERRUPT_MASK_FROM_ISR( ( x ) )
#else

/* The tick type can be read atomically, so critical sections used when the
 * tick count is returned can be defined away. */
    #define portTICK_TYPE_ENTER_CRITICAL()
    #define portTICK_TYPE_EXIT_CRITICAL()
    #define portTICK_TYPE_SET_INTERRUPT_MASK_FROM_ISR()         0
    #define portTICK_TYPE_CLEAR_INTERRUPT_MASK_FROM_ISR( x )    ( void ) ( x )
#endif /* if ( portTICK_TYPE_IS_ATOMIC == 0 ) */
```

- atomicity in the ARM architecture
    - reference: [https://developer.arm.com/documentation/ddi0406/c/Application-Level-Architecture/Application-Level-Memory-Model/Memory-types-and-attributes-and-the-memory-order-model/Atomicity-in-the-ARM-architecture?lang=en](https://developer.arm.com/documentation/ddi0406/c/Application-Level-Architecture/Application-Level-Memory-Model/Memory-types-and-attributes-and-the-memory-order-model/Atomicity-in-the-ARM-architecture?lang=en)

```markdown
Single-copy atomicity
A read or write operation is single-copy atomic if the following conditions are both true:

- After any number of write operations to a memory location, the value of the memory location is the value written by one of the write operations. 
It is impossible for part of the value of the memory location to come from one write operation and another part of the value to come from a different write operation.

- When a read operation and a write operation are made to the same memory location, the value obtained by the read operation is one of:

  - the value of the memory location before the write operation

  - the value of the memory location after the write operation.

  It is never the case that the value of the read operation is partly the value of the memory location before the write operation and partly the value of the memory
  location after the write operation.
```

- understanding why 32-bit read and write on a 32-bit architecture is always atomic
    - read and write to the memory is completely dependent on the whole operation
        - i.e. 32-bit at address 0x is determined by the write operation opj, it has nothing to do with opj-1 and opj+2
    - counter-example
        - 16-bit read and write on a 32-bit architecture
            - 32-bit at address 0x are determined by two write operations at the low and high 16-bit memories, which have mutual impact on each other
            - as for read operation, it is impacted by the write operation, see ARM’s doc.
- definition of an atomic operation
    - reference: [https://developer.arm.com/documentation/102407/0100/Atomic-operations](https://developer.arm.com/documentation/102407/0100/Atomic-operations)
    - An atomic operation is a read-modify-write sequence that is performed without interference from another requester. Like exclusive accesses in AXI, Atomic Transactions allow a requester to modify data in a particular region of memory, while ensuring that writes from other requestors do not corrupt the data
- Single instructions are always atomic on a single-core architecture
    - interrupts can only occur between instructions
- LDREX and STREX ARM instructions are used to access the memory uniquely at certain time