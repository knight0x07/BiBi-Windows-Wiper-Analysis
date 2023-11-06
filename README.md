# Analysis of BiBi-Windows Wiper targeting Israeli Organizations

On 30th October, Security Joes Incident Response team [discovered](https://www.securityjoes.com/post/bibi-linux-a-new-wiper-dropped-by-pro-hamas-hacktivist-group) a new Linux Wiper named **"BiBi-Linux"** Wiper been deployed by Pro-Hamas Hacktivist group to destroy their infrastructure. And then on November 1 2023, ESET Research [tweeted](https://twitter.com/ESETresearch/status/1719437301900595444) about a Windows version of the Bibi Wiper deployed by BiBiGun, a Hamas-backed hacktivist group that initially debuted during the 2023 Israel-Hamas conflict.

In this post, we will look at the Windows version of the BiBi Wiper known as the **"BiBi-Windows Wiper"**

## Analysis

Upon execution, the BiBi-Windows Wiper checks to see if any arguments have been passed to the BiBi Wiper. The arguments here is the directory to be destroyed - `bibi.exe <directory_to_destroyed>`. If no argument is provided, it performs the following routine for fetching the target drives

- Reads the hardcoded path: "C:\Users"
- Gets the currently available disk drives using `GetLogicalDrives()` where the return value is the bitmask, then it iterates through the A-Z (26) drives. It next does a bittest with the retrieved bitmask to determine the accessible drives on the system and appends ":\\" to the drive name.
- Here it excludes the C drive by checking for `i != 2` where 2 is the bitmask position for C drive
- Then for the available drives except the C drive it executes the `GetDriveTypeA()` which retrives the drive type, The BiBi-Windows Wiper here only targets the following drive types:

  - DRIVE_FIXED
  - DRIVE_REMOVABLE
  - DRIVE_RAMDISK

![1](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/44f0d6bd-2ea6-4d93-8f4f-208f7b73126e)

Therefore the BiBi-Windows Wiper targets
- The hardcoded directory - "C:\Users"
- And all available drives except from "C:\" drive

Further it prints the target directories on the console and retrieves the NumberOfProcessors from `GetNativeSystemInfo()` and based on the numberofprocessors it calculates the threads and then prints it onto the console.

![2](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/4828d72d-058c-4e5e-8602-a51077c45a4f)


