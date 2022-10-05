Tethered Downgrade Guide
By Mineek

discord: Mineek#6323

This tutorial was made in half an hour, its really bad but should get you started on your tethered downgrade adventure!

HUGE THANKS TO **galaxy#6181**. Without him, I wouldn't have known all this to write a guide!

# REQUIREMENTS:
- irecovery
- futurerestore
- pyimg4 (pip3 install pyimg4) (**MAKE SURE YOU UPDATED PYTHON AND NOT USING THE BUNDLED ONE!**)
- Kernel64patcher (https://github.com/iSuns9/Kernel64Patcher)
- ldid (https://github.com/ProcursusTeam/ldid)
- restored_external64_patcher (https://github.com/iSuns9/restored_external64patcher)
- asr64_patcher (https://github.com/iSuns9/asr64_patcher)
- libimg4patcher (https://github.com/iSuns9/libimg4_patcher)

**Make sure to use the forks listed above.**

# Downgrade portion:

1. Grab yourself your ipsw for iOS 15.1
2. Extract it and grab kernelcache and restore ramdisk (tip: it's the smallest .dmg in the IPSW!)
4. Extract the restore ramdisk with: `pyimg4 im4p extract -i restore_ramdisk_name.dmg -o ramdisk.dmg`
5. Mount the restore ramdisk: 


	   hdiutil attach ramdisk.dmg -mountpoint ramdisk


6. patch ASR: 
          
       asr64_patcher ramdisk/usr/sbin/asr patched_asr
        


7. Patch restored_external: 


	   restored_external64_patcher ramdisk/usr/local/bin/restored_external restored_external_patched


9. Patch libimg4: 
          
	  
       libimg4_patcher ramdisk/usr/lib/libimg4.dylib libimg4.patched
          
	  
10. Extract original entitlements from original binaries.
            
        ldid -e ramdisk/usr/local/bin/restored_external >> restored_ents.plist
        ldid -e ramdisk/usr/sbin/asr >> asr_ents.plist
            


11. Remove the old ones: 


        rm -f ramdisk/usr/sbin/asr && rm -f ramdisk/usr/local/bin/restored_external && rm -f ramdisk/usr/lib/libimg4.dylib


12. Resign the binaries: 


	       ldid -Srestored_ents.plist restored_external_patched
	       ldid -Sasr_ents.plist patched_asr
	       ldid -S libimg4.patched


13. chmod the binaries: 


	      chmod 777 restored_external_patched
	      chmod 777 patched_asr
	      chmod 777 libimg4_patched


14. Replace the original binaries with the patched ones: 


		cp -a restored_external_patched ramdisk/usr/local/bin/restored_external
		cp -a patched_asr ramdisk/usr/sbin/asr


14. Unmount the ramdisk: 


		hdiutil detach ramdisk


15. Repack the ramdisk 


		pyimg4 im4p create -i ramdisk.dmg -o ramdisk.im4p -f rdsk


16. Extract the kernel:
	

	    pyimg4 im4p extract -i kernelcache.release.* -o kernelcache.raw --extra kpp.bin 


(If your device does not have KPP which is A10 devices and up do not include `--extra kpp.bin`)

17. Patch it: 


	    Kernel64Patcher kernelcache.raw  kernelcache.patched -a


18. Repack the kernel:
(If your device does not have KPP which is A10 devices and up do not include --extra kpp.bin)


	    pyimg4 im4p create -i kernelcache.patched -o kernelcache.im4p --extra kpp.bin -f rkrn --lzss

19. Restoring the device with futurerestore:

**(MAKE SURE YOU ARE IN PWNDFU WITH SIGCHECKS REMOVED!)**


    futurerestore -t blob.shsh2 --use-pwndfu --skip-blob --rdsk ramdisk.im4p --rkrn kernelcache.im4p --custom-latest-beta --custom-latest-buildid 19H12 --latest-sep --latest-baseband ipsw.ipsw
