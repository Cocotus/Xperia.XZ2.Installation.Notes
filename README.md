# Installation of Lineage 17x (Android10) for Xperia XZ2 (Android9) using Windows computer
Following instructions describe how to achieve installation of a custom Android 10 ROM for Xperia XZ2
I like Lineage on my phones, so I choose this rom:  
https://forum.xda-developers.com/xperia-xz2/development/rom-lineageos-17-1-t4044653  

Notice that I used this guide to install it on a Xperia XZ2 dual (H8296) with stock rom Android 9 with unlocked bootloader. You can skip some parts of this guide when you have differenet prerequisite like already Android 10 installed, or bootloader already unlocked. Also I didnt provide download links to the tools and framework. You can find them yourself by googling or just use the versions I used and added in this repository. Those may be out of date soon, because I wont keep them up to date. You have been warned ;)

Credit goes to Raphos which posted the main part of the guide above. You can find the original post here:  
https://forum.xda-developers.com/showpost.php?p=82312489&postcount=268  


## Prerequisites:
- Install ADB Commandline and Fastboot tools (___platform-tools_r30.0.0-windows.zip___) on your computer: https://wiki.lineageos.org/adb_fastboot_guide.html  
I extracted the zip into folder: ___C:\adb-fastboot\___  like described in the above guide.
- Activate developers options on phone, enable USB debugging
- Download and unzip ___payload_dumper.zip___. I created folder ___C:\adb-fastboot\payload_dumper\___
- Payload dumper needs installed Python framework. So install python if necessary: https://www.python.org/downloads/windows/  
- Download the lineage image you want to use. I used the downloadlink here:
https://forum.xda-developers.com/showpost.php?p=81662883&postcount=2 and then I downloaded:
https://androidfilehost.com/?w=files&flid=304862  
___lineage-17.1-20200414-UNOFFICIAL-akari_RoW.zip___
- Download necessary OEM files for your phone model. This is also described in the offical installation notes of the lineage thread. https://forum.xda-developers.com/showpost.php?p=81662883&postcount=2  
For my modell Xperia XZ2 dual (H8296) the correct file at that time was: 
___SW_binaries_for_Xperia_Android_10.0.7.1_r1_v5a_tama___
However please download the the version which was published before the publishing date of lineage rom to avoid conflicts!!
- Get the correct TWRP file. I downloaded mine here: https://forum.xda-developers.com/showpost.php?p=82134687&postcount=2  
https://androidfilehost.com/?w=files&flid=306953   
___2020-04-13_20-20-56_twrp_sodp_xz2.tar.gz___


### Install stock Android 10 firmware using Newflasher and XperiFirm
- XperiFirm (I used: XperiFirm_5.4.0_(by_Igor_Eisberg)):   
https://forum.xda-developers.com/crossdevice-dev/sony/pc-xperifirm-xperia-firmware-downloader-t2834142  
Download newest Android 10 ROM for your device, country does not matter.  For mine: Xperia XZ2 dual (H8296) it's:  ___52.1.A.0.672 (Hongkong)___

- Newsflasher(I used: newflasher_v20):   https://forum.xda-developers.com/crossdevice-dev/sony/progress-newflasher-xperia-command-line-t3619426
- Copy newsflasher.exe into downloaded rom folder above. 
- Reboot into bootloader with command from console:  
```adb reboot bootloader```
- Start Newsflasher.exe and it should run and install. Every question I answered with "n" and enter.
After that boot into system and check if everything is installed (Android 10).

### Unlock bootloader
- Instructions: https://www.getdroidtips.com/unlock-bootloader-sony-xperia-xz2/
- With your code you can now unlock bootloader via fastboot with command from console:  
```adb reboot bootloader```  
```fastboot oem unlock YOURCODE```

### Preparing LINEAGE files with payload dumper:
- Extract the zipped lineage image and move the payload.bin to folder C:\adb-fastboot\payload_dumper     
- Use following commands from console:  
```cd C:\adb-fastboot\payload_dumper\```  
```python -m pip install -r C:\adb-fastboot\payload_dumper\requirements.txt```    
```python payload_dumper.py payload.bin```    

This will put the files in payload_dumper/output/ folder: boot, dtbo, system, vbmeta and vendor.img  

### Installation of LINEAGE:
- Use following commands from console:  
```fastboot erase boot_a```  
```fastboot erase cache```  
```fastboot erase system_a```  
```fastboot erase userdata```  
```fastboot erase boot_b```  
```fastboot erase system_b```  
```fastboot flash boot C:\adb-fastboot\payload_dumper\output\boot.img```  
```fastboot flash dtbo C:\adb-fastboot\payload_dumper\output\dtbo.img```  
```fastboot flash system C:\adb-fastboot\payload_dumper\output\system.img```  
```fastboot flash vbmeta C:\adb-fastboot\payload_dumper\output\vbmeta.img```  
```fastboot flash vendor C:\adb-fastboot\payload_dumper\output\vendor.img```  
```fastboot erase userdata```  
- Now I put the OEM img file also into the output folder and used following commands:  
```fastboot flash oem_a C:\adb-fastboot\payload_dumper\output\SW_binaries_for_Xperia_Android_10.0.7.1_r1_v5a_tama.img``` 
```fastboot flash oem_b C:\adb-fastboot\payload_dumper\output\SW_binaries_for_Xperia_Android_10.0.7.1_r1_v5a_tama.img``` 
```fastboot reboot-bootloader```  
- Now we are going to boot from TWRP to install/flash zip files! For that I put the extracted twrp package into C:\adb-fastboot\payload_dumper\output\twrp_sodp\twrp-xz2.img and used commands:  
```fastboot --disable-verity --disable-verification flash vbmeta C:\adb-fastboot\payload_dumper\output\twrp_sodp\vbmeta.img```  
```fastboot boot C:\adb-fastboot\payload_dumper\output\twrp_sodp\twrp-xz2.img```  

- That will start the TWRP. In TWRP do the following installations:
1. flash sony-dualsim-patcher-v4.zip
2. flash newest magisk.zip
3. reboot system

DONE! 
