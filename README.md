# Technical Analysis of BiBi-Windows Wiper Targeting Israeli Organizations

On 30th October, Security Joes Incident Response team [discovered](https://www.securityjoes.com/post/bibi-linux-a-new-wiper-dropped-by-pro-hamas-hacktivist-group) a new Linux Wiper named **"BiBi-Linux"** Wiper been deployed by Pro-Hamas Hacktivist group to destroy their infrastructure. And then on November 1 2023, ESET Research [tweeted](https://twitter.com/ESETresearch/status/1719437301900595444) about a Windows version of the Bibi Wiper deployed by BiBiGun, a Hamas-backed hacktivist group that initially debuted during the 2023 Israel-Hamas conflict.

In this post, we will look at the Windows version of the BiBi Wiper known as the **"BiBi-Windows Wiper"**

## Technical Analysis

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

Now lets understand how BiBi-Windows Wiper destroys the data on the machine.

- The Wiper firsty walks through the directory to be encrypted recursively and then retrives the file size of the target files.
- Then it executes the wiper function in loop against the target files based on the file size, where the second argument to the wiper function is "0xFF"
- The Wiper function implements the `Mersenne Twister PseudoRandom Number Generator Algorithm` which generates random numbers as shown below.

![5](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/976436bf-a95b-40e3-9f57-a1187a04006b)

- The random number generated from the function `sub_140008BF0()` then performs modulus (%) with the value "0xFF + 1 = 100" and the output (remainder) of the modulus operation is the byte which is been overwritten in the target file byte by byte.

![7](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/fd0acb14-f92b-46d4-865e-9be23920ee8f)

Example below against a file consisting of the data "hello" and is been overwritten by these 5 random bytes in a loop.

**ogfile_val -> rand_num % (hardcoded_val + 1) -> overwrite_val**

```    
h - 0xDD9B40A5 % 0x100 -> 0xA5
e - 0x37905317 % 0x100 -> 0x17
l - 0x54B7A20C % 0x100 -> 0x0C
l - 0xDBD10533 % 0x100 -> 0x33
o - 0x18ED3C42 % 0x100 -> 0x42
```
File Overwritten with Random bytes - **Destroyed!**

![9](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/460563a8-5f4d-4482-b904-a79074d77858)

Now once the data in the file is overwritten with random bytes and the file is been destroyed, the BiBi Wiper changes the name of the file in the following manner:

- It calls the previous Mersenne Twister function again where the hardcoded argument this time is "0x3D". So once the random number is generated it performs modulus with "0x3E" and the output (remainder) value is stored in the similar manner.
- Further the output remainder value is been multiplied by 2 and then the value is indexed against a wide string as represented below

```
rand_no = twisterfunc()
remainder_output = rand_no % (0x3D + 1)
wide_string = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
filename_byte = wide_string[remainder_output * 2]

Thus,
if random num = 0xC6D15D79
mod_out = 0xC6D15D79 % 0x3E => 0x31
offset = 0x31 × 0x2 = 0x62
so the 62nd indexed value in the wide string becomes one of the character for the filename. **In this case "n"**

```
The same routine is been executed 0xA (10) times and all the indexed values are appended together forming the random file name eg. **1wnRvB6teT** and then the extension **".BiBi" (Bibi is a nickname used for Israel's Prime Minister, Benjamin Netanyahu)** is added along the **round number at the end** in the following manner - **<rand_filename>.BiBi<roundno>**

![10](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/7a1167a3-3f47-4b46-911d-eb61ca6500dd)

Therefore in the following manner the BiBi-Windows Wiper would destroy all the files in the target directories by overwriting the data in the files with random bytes, now as the loop i.e the rounds are till infinity the Wiper will keep on overwriting the files multiple time recursively with random bytes till infinity and wont stop thus destroying the files on the machine completely!

BiBi-Windows Wiper execution showcasing the Target directory, CPU Cores, Threads, Round Number, Stats, and destroyed file with .BiBi extension

![12](https://github.com/knight0x07/BiBi-Windows-Wiper-Analysis/assets/60843949/6221f60a-5b3d-41d3-b6e5-4f9dd02834ea)


Thanks for reading! - knight0x07






