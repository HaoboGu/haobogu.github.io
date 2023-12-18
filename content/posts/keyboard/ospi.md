# openocd配置

 在openocd里面配置ospi，只能直接写寄存器。因此需要首先了解一下寄存器的配置

## ospi1的起始位置是

0x58005000

## OCTOSPI_CR寄存器

CR即control register。是ospi最基础的配置。

Address offset: 0x0000 Reset value: 0x0000 0000

一共32Bit

其中比较重要的：

- FMODE：28-29，配置了ospi的模式，有
  - 00： Indirect-write mode 
  - 01: Indirect-read mode 
  - 10: Automatic status-polling mode 
  - 11: Memory-mapped mode

在使用ospi flash的时候，使用11

配置：

```shell
mww 0x52005000 0x30400003         ;# OCTOSPI_CR: FMODE=0x11, APMS=1, FTHRES=0, FSEL=0, DQM=0, TCEN=1
```



## (OCTOSPI_DCR1

Device configuration register 1，设备配置寄存器1

```shell
mww 0x52005008 0x01160100				;# OCTOSPI_DCR1: MTYP=0x1, FSIZE=0x19, CSHT=0x01, CKMODE=0, DLYBYP=0
```

MTYP: 001: Macronix mode

DEVSIZE: Number of bytes in device = 2[DEVSIZE+1]. 因此，在我们这里使用8M的情况下，应该是2^23，即devsize=22=0x16

## CTOSPI communication configuration register (OCTOSPI_CCR)

DMODE: 011 data 4line mode

