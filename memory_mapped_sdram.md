# Memory Mapping of SDRAM on STM32: A Detailed Explanation
In STM32 microcontrollers, SDRAM (Synchronous Dynamic Random Access Memory) is interfaced with the microcontroller through a peripheral known as FMC (Flexible Memory Controller). When the FMC is configured to communicate with SDRAM, it maps the SDRAM directly into the microcontroller's memory space. This means that the SDRAM behaves like part of the microcontroller’s memory and can be accessed like any other memory (e.g., RAM or flash), using simple memory read and write operations.

Let's break this down step by step:

1. What is Memory Mapping?
Memory mapping is the process of assigning a physical address to a peripheral, storage, or memory module. When SDRAM is memory-mapped, the FMC assigns it to a specific range of addresses in the microcontroller's memory address space.

For example, in STM32H7, SDRAM is typically mapped starting at an address like 0xC0000000. This base address is assigned by the FMC controller, and the size of the SDRAM determines the length of the address range.

Once mapped, you can access this address range just like accessing normal RAM. For example, if you want to store data at an address in SDRAM, you can simply write to that address, and to read from it, you just read from the address.

2. Why You Don’t Need HAL APIs for Read/Write?
Because the SDRAM is memory-mapped, there is no need to use dedicated functions (such as HAL APIs) to handle reads and writes. You can use simple C pointer dereferencing or standard library functions like memcpy to access the SDRAM.

## Example 1: Pointer Dereferencing for Accessing SDRAM
Let’s assume the SDRAM starts at address 0xC0000000. You can directly read and write data using a pointer that points to this address.

```c
Copier le code
#define SDRAM_BASE_ADDRESS  0xC0000000  // SDRAM Base address

// Write data to SDRAM
void WriteToSDRAM(uint32_t address_offset, uint32_t data) {
    uint32_t *sdram_address = (uint32_t *)(SDRAM_BASE_ADDRESS + address_offset);
    *sdram_address = data;  // Directly write data to SDRAM
}

// Read data from SDRAM
uint32_t ReadFromSDRAM(uint32_t address_offset) {
    uint32_t *sdram_address = (uint32_t *)(SDRAM_BASE_ADDRESS + address_offset);
    return *sdram_address;  // Directly read data from SDRAM
}

int main() {
    WriteToSDRAM(0x0000, 0x12345678);  // Write at base address
    uint32_t data = ReadFromSDRAM(0x0000);  // Read from base address
}
```
In this example:

You define a pointer that points to a location in SDRAM (SDRAM_BASE_ADDRESS + address_offset).
You can dereference that pointer to read or write data, just as you would with normal RAM.

## Example 2: Using memcpy to Read/Write SDRAM
If you need to transfer large blocks of data to or from SDRAM, you can use memcpy() to copy data between a buffer in regular memory and SDRAM.

```c
Copier le code
#define SDRAM_BASE_ADDRESS  0xC0000000  // SDRAM Base address

// Write block of data to SDRAM
void WriteBlockToSDRAM(uint32_t address_offset, uint32_t *src_buffer, uint32_t size) {
    uint32_t *sdram_address = (uint32_t *)(SDRAM_BASE_ADDRESS + address_offset);
    memcpy(sdram_address, src_buffer, size * sizeof(uint32_t));  // Copy data to SDRAM
}

// Read block of data from SDRAM
void ReadBlockFromSDRAM(uint32_t address_offset, uint32_t *dest_buffer, uint32_t size) {
    uint32_t *sdram_address = (uint32_t *)(SDRAM_BASE_ADDRESS + address_offset);
    memcpy(dest_buffer, sdram_address, size * sizeof(uint32_t));  // Copy data from SDRAM
}

int main() {
    uint32_t write_data[100] = {0};  // Data to write
    uint32_t read_data[100] = {0};   // Buffer to store read data

    // Write data block to SDRAM
    WriteBlockToSDRAM(0x0000, write_data, 100);

    // Read data block from SDRAM
    ReadBlockFromSDRAM(0x0000, read_data, 100);
}
```
3. Hardware Efficiency: No Need for Extra Software Layers
Using memory-mapped SDRAM and directly accessing it provides significant benefits:

No Software Overhead: The HAL read/write functions (HAL_SDRAM_Read_8b, HAL_SDRAM_Write_8b, etc.) essentially use memory copying in software. By directly accessing SDRAM, you avoid the unnecessary overhead of additional function calls and redundant logic.

Simpler Access: You directly manipulate SDRAM like any other part of the MCU’s memory. No special function calls are required, and you can use standard programming techniques to handle memory.

Hardware-Accelerated Transfers: Since the FMC controller manages the interface with SDRAM, you benefit from hardware-accelerated access speeds. There’s no need to interact with low-level details such as timing or control signals, as FMC takes care of this.

4. Use Case in High-Performance Applications
In applications that need fast memory operations, directly accessing SDRAM as memory-mapped space offers performance advantages:

Speed: Accessing SDRAM directly via pointers or memcpy is faster than using HAL software APIs.
Efficiency: It minimizes the CPU load, especially when combined with DMA for transferring large blocks of data.
Simplicity: The code is simpler to read and maintain, as you avoid using specialized functions for basic memory operations.

5. When HAL Functions Might Still Be Useful
While the read/write functions might seem redundant, the HAL APIs offer higher-level abstraction and could be useful in certain cases:

Portability: If your application needs to run across different STM32 families or external memory types, the HAL APIs provide a consistent interface.
Error Handling: The HAL functions include built-in error handling (e.g., checking SDRAM state), so they might be useful if you need additional checks in your application.
Conclusion
The key takeaway is that SDRAM is memory-mapped in STM32 systems via the FMC, making it directly accessible just like internal RAM or flash memory. This means that, for many use cases, you don't need the overhead of dedicated HAL APIs to read and write data. You can directly manipulate memory using normal C memory operations (memcpy, pointer dereferencing), which leads to simpler and more efficient code.

For advanced use cases, DMA can be used for bulk data transfers, further offloading the CPU and improving performance.