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

Further it creates a new thread which reads the commands stored in reverse, & then creates a new process using `CreateProcessA` to execute those commands. Following are the commands

- cmd.exe /c bcdedit /set {default} recoveryenabled no - Disables Windows Recovery Environment 
- cmd.exe / c bcdedit / set {default} bootstatuspolicy ignoreallfailures - Force the system to boot normally rather than into the Windows Recovery Environment
- cmd.exe /c wmic shadowcopy delete - Delete Volume Shadow Copies using WMIC
- cmd.exe /c  vssadmin delete   shadows /quIet /all - Delete Volume Shadow Copies using VssAdmin

![3](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/c345a18d-df95-4515-9a0d-db075e7c3080)

Furthermore it creates another thread which executes of the Main Wiper routines. The Wiper routines perform the following actions

- The Arguments to the Wiper Function are:
	- Arg1 - Path of the Directory to be destroyed (Could be provided by the Operator or retrived as explained before)
	- Arg2 - Number of threads
- Then it initiates an infinite loop where the counter is the Round "[+] Round %d\n" value - therefore once the Wiper is executed it would keep destroying the data infinitely!
- Further based on the number of threads, it creates multiple threads in a loop which execute the main Wiper function.
- **The BiBi-Windows wiper excludes the files with ".exe", ".dll" and ".sys" extension**

![4](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/cd0ea284-eb37-413b-a245-78d81d759382)

### Wiper Function Analysis

Now lets understand how the BiBi-Windows Wiper destroys the data on the machine.


