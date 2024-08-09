

### 读写固定端口代码
```c
#include <stdio.h>
#include <stdint.h>
 
// 配置地址寄存器的IO端口地址
#define PCI_CONFIG_ADDRESS 0xCF8
// 配置数据寄存器的IO端口地址
#define PCI_CONFIG_DATA 0xCFC
 
// 函数：写入配置地址寄存器
void write_config_address(uint32_t address) {
    // 由于是IO端口操作，需要用特定的指令或函数
    // 这里假设有一个write_io_32函数来写入32位数据到IO端口
    write_io_32(PCI_CONFIG_ADDRESS, address);
}
 
// 函数：从配置数据寄存器读取数据
uint32_t read_config_data() {
    // 由于是IO端口操作，需要用特定的指令或函数
    // 这里假设有一个read_io_32函数来从IO端口读取32位数据
    return read_io_32(PCI_CONFIG_DATA);
}
 
// 函数：写入配置数据寄存器
void write_config_data(uint32_t data) {
    // 由于是IO端口操作，需要用特定的指令或函数
    // 这里假设有一个write_io_32函数来写入32位数据到IO端口
    write_io_32(PCI_CONFIG_DATA, data);
}
 
int main() {
    // 示例：读取PCI设备的配置空间的某个地址的数据
    uint32_t address = /* 设定你要访问的PCI设备的配置地址 */;
    uint32_t data;
 
    // 写入配置地址寄存器
    write_config_address(address);
    // 读取配置数据寄存器
    data = read_config_data();
 
    printf("读取的数据是: 0x%08X\n", data);
 
    // 示例：写入PCI设备的配置空间的某个地址的数据
    data = /* 设定你要写入的数据 */;
 
    // 写入配置地址寄存器
    write_config_address(address);
    // 写入配置数据寄存器
    write_config_data(data);
 
    return 0;
}

```


## 通过IO端口来访问
参考博文：https://blog.csdn.net/faxiang1230/article/details/124436795
### 读写配置空间示例
```c
uint16_t pciConfigReadWord(uint8_t bus, uint8_t slot, uint8_t func, uint8_t offset) {
    uint32_t address;
    uint32_t lbus  = (uint32_t)bus;
    uint32_t lslot = (uint32_t)slot;
    uint32_t lfunc = (uint32_t)func;
    uint16_t tmp = 0;
 
    // Create configuration address as per Figure 1
    address = (uint32_t)((lbus << 16) | (lslot << 11) |
              (lfunc << 8) | (offset & 0xFC) | ((uint32_t)0x80000000));
 
    // Write out the address
    outl(0xCF8, address);
    // Read in the data
    // (offset & 2) * 8) = 0 will choose the first word of the 32-bit register
    tmp = (uint16_t)((inl(0xCFC) >> ((offset & 2) * 8)) & 0xFFFF);
    return tmp;
}
void pciConfigWriteWord(uint8_t bus, uint8_t slot, uint8_t func, uint8_t offset, uint32_t val) {
    uint32_t address;
    uint32_t lbus  = (uint32_t)bus;
    uint32_t lslot = (uint32_t)slot;
    uint32_t lfunc = (uint32_t)func;
    uint16_t tmp = 0;
 
    // Create configuration address as per Figure 1
    address = (uint32_t)((lbus << 16) | (lslot << 11) |
              (lfunc << 8) | (offset & 0xFC));
	// Write out data
 	outl(0xCFC, val);
    // Write out the address
    outl(0xCF8, address);
}
```
## 通过MMIO内存映射来访问（又称作内存空间）
### 读写配置空间示例
对于不支持ioport的设计，芯片在划分外设地址空间时会留出来一部分地址空间给PCI配置空间，根据偏移来访问对应的地址空间，这个就需要 host bridge 识别地址空间，将访问 PCI 转发到设备上。

对于PCI的地址计算如下:

```
Physical_Address = MMIO_Starting_Physical_Address + ((bus - MMIO_Starting_Bus) << 20 | device << 15 | func << 12) + offset
```






