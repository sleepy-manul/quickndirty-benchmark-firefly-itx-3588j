Disclaimer / warnings:
- I am NOT an industry professional in benchmarking storage. 
  The test case was constructed to the best of my abilities without guarantees or warranties. YMMV.
  Please do NOT use this report to make important business decisions. This report is meant to give
  a rough indication of the possible performance and maybe inspire you to conduct your own evaluation and testing.
- I cannot exclude the possibility of mistakes, but I can give the assurance that no effort was made to influence the benchmark 
  results in favour or disfavour of the hardware discussed in this report.
- I have not been paid by or received any benefits or favours from any of the manufacturers or vendors mentioned in this report. 
  All hardware was purchased with my private funds. All software used was provided by Firefly as part of their
  board support package (BSP) and/or is available under Open Source Software (OSS) licenses.
- The following information was redacted from the hwinfo output for privacy reasons:
  - serial numbers
  - MAC and IPv4/IPv6 addresses
- The software versions are in constant flux as I run a rolling Linux distribution (openSUSE Tumbleweed).
  At the time of the test, the software versions of the core OS packages were:
  - Linux Kernel "5.10.160" (actually, an Android OS 12 kernel heavily patched by Rockchip and supplied in the BSP).
    The kernel was recompiled to fit into openSUSE's standard kernel packaging. Sources:
    https://build.opensuse.org/package/show/home:sleepy-manul:ARM:Factory:Contrib:Firefly-ITX-3588J/kernel-firefly-itx-3588j:kernel-fireflyitx3588j
  - C runtime library: glibc-2.39-4.2.aarch64
  - Benchmarking software: iozone-3.506-benchmark.32.22.aarch64
  
Notes on btrfs:
- btrfs is a copy-on-write (CoW) filesystem. That means when modifying an existing file, any changed blocks will first be 
  written to a different block; the original blocks will not be affected/freed until the writing process has been completed.
- While this is great for file system consistency, it affects the results of benchmarks and use cases where many changes are 
  written frequently to huge files (e.g. databases, virtualization images, ...). It also makes the "rewrite" tests of iozone
  pointless since that test works under the explicit assumption that it is overwriting blocks in an existing file.
- Because of this, all tests on a btrfs volume are done in a directory explicitly marked as non-COW, like this:
```
    /mnt/emmc # mkdir no_data_copy_on_write
    /mnt/emmc # chattr +C no_data_copy_on_write
    /mnt/emmc # cd no_data_copy_on_write/
    /mnt/emmc/no_data_copy_on_write # iozone -a ...
```

iozone invocation:
- The description of the tests run by iozone is here: https://www.iozone.org/docs/IOzone_msword_98.pdf . The descriptions of the command line options are from the iozone manpage.
- iozone -a -+u -I -M -e -s 1g
  * -a     Used to select full automatic mode. Produces output that covers all tested file operations for record sizes of 4k to 16M for file sizes of 64k to 512M.
  * -+u    Used to enable CPU statistics collection.
  * -I     Use  DIRECT  IO  if  possible  for all file operations. Tells the filesystem that all operations to the file are to bypass the buffer cache and go directly to disk. (not available on all platforms)
  * -M     Iozone will call uname() and will put the string in the output file.
  * -e     Include flush (fsync,fflush) in the timing calculation
  * -s #   Used to specify the size, in Kbytes, of the file to test. One may also specify -s #k (size in Kbytes) or -s #m (size in Mbytes) or -s #g (size in Gbytes).
    
Testcase:
- Mainboard: Firefly ITX-3588J ( https://en.t-firefly.com/product/industry/itx3588j )
  SKU: 32 GB RAM / 256 GB eMMC
  - eMMC data provided by "mmc extcsd read /dev/mmcblk0":
	  - Card Type [CARD_TYPE: 0x57]
	  - HS400 Dual Data Rate eMMC @200MHz 1.8VI/O
	  - HS200 Single Data Rate eMMC @200MHz 1.8VI/O
	  - HS Dual Data Rate eMMC @52MHz 1.8V or 3VI/O
	  - HS eMMC @52MHz - at rated device voltage(s)
	  - HS eMMC @26MHz - at rated device voltage(s)
	  - High-speed interface timing [HS_TIMING: 0x03]
	  - Enhanced Strobe mode [STROBE_SUPPORT: 0x01]
	  - Vendor: unknown, device identifies as "Y0S256"
  - Filesystem: btrfs
  - Cooling: the standard cooler + fan included with the board.
  - Results file: iozone-emmc-run.log
  
- For m2/SATA test:
  - SSD: Transcend TS1TMTS425S ( https://de.transcend-info.com/product/internal-ssd/mts425s )
  - Note: Although this SSD has an m.2 form factor, it is NOT an NVMe SSD. Instead, the interface is SATA 3.0 (because this is the only protocol supported by the Firefly-ITX-3588J for the m.2 storage slot), which limits the maximum throughput to a theoretical 600 MB/sec.
  - Filesystem: btrfs
  - Results file: iozone-m2_transcend-run.log
  - Cooling: none. Cooling is most likely not required, as the following was shown in "smartctl -x /dev/sdb" after extensive benchmarking:
```
SCT Status Version:                  3
SCT Version (vendor specific):       0 (0x0000)
Device State:                        Active (0)
Current Temperature:                    48 Celsius
Power Cycle Min/Max Temperature:     48/48 Celsius
Lifetime    Min/Max Temperature:     23/56 Celsius
Specified Max Operating Temperature:   100 Celsius
Under/Over Temperature Limit Count:   0/0
```

- For the PCIe/NVMe adapter test:
  - NVMe SSD:
    SSKT-070 Kingston NV2 NVMe, PCIe 4.0 M.2 Typ 2280 - 1 TB 
    Device model identification: KINGSTON SNV2S1000G
    (Note: The PCIe slot in the Firefly-ITX-3588J only supports PCIe v3.0, which limits the maximum bandwidth)
    
  - Adapter card:
    Graugear PCIe card for M.2 NVMe SSD to PCIe 4.0 x4
    https://graugear.de/portfolio-item/g-m2pci01/
    (Note: The PCIe slot in the Firefly-ITX-3588J only supports PCIe v3.0, which limits the maximum bandwidth)
  - Cooling: Passive heatsink
  - File system: btrfs
  - Results file: iozone-nvme-kingston-run.log  

 
