# SDRAM peripheral brainstorming to simple re-use

# Advantages of Combining Global and Unitary APIs

## Global Configuration

- **Ease of Use**: Users can easily configure the SDRAM by setting all necessary parameters at once during initialization.
- **Efficiency**: It ensures a simple and clean setup process, particularly for applications where SDRAM settings do not need to change after the initial configuration.
- **Suitable for Most Applications**: For many use cases, once the SDRAM is configured, it remains constant throughout the lifetime of the application.

### Example of Global SDRAM Configuration

```c
#include "stm32f7xx_hal.h"

// Declare SDRAM handle and configuration structures
SDRAM_HandleTypeDef hsdram;
FMC_SDRAM_TimingTypeDef SDRAM_Timing;
FMC_SDRAM_InitTypeDef SDRAM_MemConfig;

void SDRAM_GlobalInit(void)
{
    // Initialize SDRAM Memory Config
    SDRAM_MemConfig.SDBank             = FMC_SDRAM_BANK1;
    SDRAM_MemConfig.ColumnBitsNumber    = FMC_SDRAM_COLUMN_BITS_NUM_8;   // 8 bits
    SDRAM_MemConfig.RowBitsNumber       = FMC_SDRAM_ROW_BITS_NUM_11;     // 11 bits
    SDRAM_MemConfig.MemoryDataWidth     = FMC_SDRAM_MEM_BUS_WIDTH_16;    // 16-bit bus
    SDRAM_MemConfig.InternalBankNumber  = FMC_SDRAM_INTERN_BANKS_NUM_4;  // 4 banks
    SDRAM_MemConfig.CASLatency          = FMC_SDRAM_CAS_LATENCY_3;       // CAS latency 3
    SDRAM_MemConfig.WriteProtection     = FMC_SDRAM_WRITE_PROTECTION_DISABLE;
    SDRAM_MemConfig.SDClockPeriod       = FMC_SDRAM_CLOCK_PERIOD_2;      // Clock period 2
    SDRAM_MemConfig.ReadBurst           = FMC_SDRAM_RBURST_ENABLE;       // Enable read burst
    SDRAM_MemConfig.ReadPipeDelay       = FMC_SDRAM_RPIPE_DELAY_1;       // 1 cycle delay

    // Initialize SDRAM Timing Config
    SDRAM_Timing.LoadToActiveDelay    = 2;
    SDRAM_Timing.ExitSelfRefreshDelay = 7;
    SDRAM_Timing.SelfRefreshTime      = 4;
    SDRAM_Timing.RowCycleDelay        = 7;
    SDRAM_Timing.WriteRecoveryTime    = 3;
    SDRAM_Timing.RPDelay              = 2;
    SDRAM_Timing.RCDDelay             = 2;

    // Initialize SDRAM with global settings
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle initialization error
        Error_Handler();
    }
}
```
## Unitary Configuration
Runtime Flexibility: Users can adjust specific parameters dynamically at runtime without resetting the entire configuration, making the SDRAM interface more adaptable.
Granular Control: Provides advanced users with the ability to control specific parameters (e.g., CAS latency, column bits) for optimization, based on the application’s current requirements, like switching between low-power and high-performance modes.
Perfect for Dynamic Systems: For applications that work with external memories or need to change SDRAM configurations depending on different operating modes (e.g., high performance vs. low power), unitary APIs provide this critical capability.

Example of Unitary API for Memory Configuration

```c
Copier le code
void SDRAM_SetCASLatency(uint32_t new_cas_latency)
{
    SDRAM_MemConfig.CASLatency = new_cas_latency;

    // Reinitialize SDRAM with updated CAS latency
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle error
        Error_Handler();
    }
}

void SDRAM_SetDataWidth(uint32_t data_width)
{
    SDRAM_MemConfig.MemoryDataWidth = data_width;

    // Reinitialize SDRAM with updated data bus width
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle error
        Error_Handler();
    }
}

void SDRAM_SetColumnBits(uint32_t column_bits)
{
    SDRAM_MemConfig.ColumnBitsNumber = column_bits;

    // Reinitialize SDRAM with updated column bits number
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle error
        Error_Handler();
    }
}
```

Example of Unitary API for Timing Configuration

```c
Copier le code
void SDRAM_SetRowCycleDelay(uint32_t delay)
{
    SDRAM_Timing.RowCycleDelay = delay;

    // Reinitialize SDRAM with updated timing
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle error
        Error_Handler();
    }
}

void SDRAM_SetSelfRefreshTime(uint32_t time)
{
    SDRAM_Timing.SelfRefreshTime = time;

    // Reinitialize SDRAM with updated timing
    if (HAL_SDRAM_Init(&hsdram, &SDRAM_MemConfig, &SDRAM_Timing) != HAL_OK)
    {
        // Handle error
        Error_Handler();
    }
}
```
### Key Benefits for SDRAM and External Memory Management
#### Best of Both Worlds
- By offering both global and unitary APIs, you cater to both novice users (who want simple initialization) and advanced users (who need to tweak specific parameters).
Novice users can stick with the global configuration, while more experienced developers can use the unitary APIs to modify settings as needed.
Better Performance

- The ability to fine-tune SDRAM at runtime helps optimize the system for different use cases. For example, an application can adjust the CAS latency to match external memory characteristics or switch memory configurations when working with different devices.
User Experience

- Offering this level of flexibility enhances the user experience by giving them the freedom to adapt SDRAM performance and behavior to their needs.
It also improves compatibility with external memories by adjusting timing and memory parameters to match the specific requirements of external SDRAM or other connected memory components.
## Practical Example
### Imagine a scenario where an embedded system switches between different modes of operation—such as deep sleep mode, active data processing mode, or external memory interface. With both global and unitary APIs, you can manage SDRAM configuration effectively:

- Global Configuration: When the system boots up, the SDRAM can be initialized with default timing and memory parameters using a global configuration.

- Unitary Configuration:

When switching to data processing mode, the application might need to optimize for speed, increasing the CAS latency or expanding the data bus width.
When switching to low-power mode, the timing parameters and memory width might need to be adjusted to reduce power consumption.
If working with an external memory (such as Flash or another SDRAM chip), users could modify the SDRAM interface timing to optimize communication with that specific memory device.
- Example of Switching Between High-Performance and Low-Power Modes
```c
Copier le code
void SwitchToHighPerformanceMode(void)
{
    // Set CAS latency to 3 (higher latency, more performance)
    SDRAM_SetCASLatency(FMC_SDRAM_CAS_LATENCY_3);

    // Set the data bus width to 32 bits for maximum throughput
    SDRAM_SetDataWidth(FMC_SDRAM_MEM_BUS_WIDTH_32);

    // Adjust other timing settings for performance
    SDRAM_SetRowCycleDelay(7);  // Longer row cycle delay
    SDRAM_SetSelfRefreshTime(5);  // Self-refresh timing for high performance
}

void SwitchToLowPowerMode(void)
{
    // Set CAS latency to 2 (lower latency, less power consumption)
    SDRAM_SetCASLatency(FMC_SDRAM_CAS_LATENCY_2);

    // Set data bus width to 16 bits to save power
    SDRAM_SetDataWidth(FMC_SDRAM_MEM_BUS_WIDTH_16);

    // Adjust timing settings for low power
    SDRAM_SetRowCycleDelay(4);  // Shorter row cycle delay
    SDRAM_SetSelfRefreshTime(3);  // Reduced self-refresh time for power savings
}
```
- Example of Managing SDRAM Timing for External Memory Interface
```c
Copier le code
void OptimizeForExternalMemory(void)
{
    // Adjust timing for external memory interface
    SDRAM_SetRowCycleDelay(9);  // Increase row cycle delay to match external memory timing
    SDRAM_SetSelfRefreshTime(6);  // Adjust self-refresh timing for external memory

    // Optionally change memory configuration if needed
    SDRAM_SetCASLatency(FMC_SDRAM_CAS_LATENCY_3);  // Higher CAS latency for external memory compatibility
}
```
# Conclusion

- By providing both global and unitary APIs for SDRAM timing and memory configurations:

- You make the SDRAM interface adaptable for a wide range of applications.
- The system can be optimized dynamically, enhancing both performance and power efficiency.
- You offer granular control for those who need it, while keeping things simple for those who prefer a global configuration.
- This combination will improve the experience for users, especially those working with external memories in embedded systems, making SDRAM management easier and more efficient.