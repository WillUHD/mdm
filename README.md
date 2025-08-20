# Bypassing MDM Enrollment using Monterey Exploit

## 0. Requirements: 
   - a Mac that can run macOS Monterey
     > Basically any Mac released within the years 2015-2022. This guide will specifically target Intel T2/M-series Macs since they have the highest level of security protection. 
   - a macOS Monterey bootable installer (must be external)
     > Monterey is explicitly necessary because we will be using a bypass that's patched in later versions
   - a temporary internet connection (can be hotspot)
     > The bypass involves turning off networks, so make sure you are able to power it off instantly! Sometimes, turning a WiFi access point off will not disconnect immediately, so preferably power-off if you can. 

## 1. Prepare the installation
 1. make sure you are ready to wipe all conntents of your disk! 
 2. plug in the bootable USB, and say one last goodbye to your current macOS installation before restarting
 3. press and hold the `option` key during startup (or the Apple silicon equivalent) until you see a **boot selection screen**. Select the bootable installer. 
 4. if RecoveryOS prompts you to connect to a network, **this is likely Activation Lock**. Otherwise, enter the password for an administrator of the computer (if prompted) and proceed. 
 5. go to Disk Utility and wipe your disk. **APFS, GUID partition map**. 
 6. if you're not already, you **should** be connected to an internet! 

## 2. Install macOS
1. Quit Disk Utility to return to **Recovery main menu**. 
2. **Enter the macOS installer app**, agree to license, and install to the drive you just formatted. 
3. The progress bar will complete and automatically restart. The disk copy phase has completed. 
4. Now, you will likely see a **black screen with an Apple logo** in the center and a progress bar captioned, *"About X minutes remaining..."*. 
   - **DO NOT LEAVE YOUR MAC ALONE BY NOW. MAKE SURE YOU HAVE COMPLETE ACCESS TO YOUR NETWORK SWITCH!** 
   - may that be router's power source, or phone hotspot, make sure you have access to it!
5. Observe and wait for the progress bar to hit exactly "*About 2 minutes remaining...*"
   - The progress bar is known to be unreliable at times! It may jump from "17 minutes remaining" to 3 in a sudden, and it also isn't proportional to the time remaining of the install! 
   - So don't try to time the progress bar!  
   - **DO NOT MESS UP FOR WHAT'S COMING UP BELOW!!!**. 
6. Keep a CLOSE eye, and as soon as the progressbar hits "*About one minute remaining...*", disconnect all nearby available networks previously connected/known to by this computer via the RecoveryOS. 
   - **You have a limited time frame, which is until the progress bar finishes, to turn off your internet source**. 
   - If you fail this step, you have to start over. 
   - You don't have to turn off ALL visible internet nearby your Mac. Only the locations you've previously connected to in te past RecoveryOS session (where you wiped your disk). 

## 3. Setup macOS
 1. Now, your internet **should stay off** and you **should be seeing the Setup screen**. If you're not seeing the setup screen, your installation is corrupted, which *could* mean that you did something with the internet (but not in all cases).
 2.  Proceed the Setup as usual, while **keeping your internet off**. 
 3.  If you are prompted with the choice to **enroll an MDM profile**, **your installation is cooked and you must start over again**. There is nothing corrupted with the installer, you made a mistake by trying to time the installer. **There is no known way to fix this except reinstalling**. 
 4. If you don't see the choice to enroll and instead are met with a page to **select WiFi**, congrats, your installation isn't cooked. Select the *"More Options"* or "*Advanced*" label in the bottom left and select *"My computer does not connect to the internet"*. **DO NOT CONNECT TO WIFI HERE!!!** 
     - The whole idea is to leverage the bug that occurs right after the OS finishs installing, but doesn't readily connect to internet, bypassing the security check of MDM profiles and making it believe you're safe. So it's very important to STAY offline and not give it any chances to enroll you. 
 5. Proceed to create a user and do normal user stuff, until you see the Monterey desktop with menubar and dock. Stay disconnected. 
 6. Unplug your external installation media. 

## 4. Turn off `csrutil`
1. Shut down your Mac and hold down `cmd + r` (or the Apple silicon equivalent) to go to Recovery. Note that this is NOT the exteral installer RecoveryOS, this is the native Recovery on your Mac. 
2. Enter the password for an admin if necessary to proceed to Recovery menu. 
3. Press `cmd + shift + t`, or go in the menubar Utilities > Terminal. 
4. Type in this exactly: `csrutil disable`
   - This turns off System Integrity Protection, and allows us to modify the core system files later on. 
   - This won't harm your Mac. 
5. Reboot your Mac to normal (don't go to Recovery again). 

## 5. Block MDM Re-enroll
1. Go to the Terminal app and type exactly these lines: 
    ```zsh
    echo "0.0.0.0 iprofiles.apple.com" | sudo tee -a /etc/hosts
    echo "0.0.0.0 mdmenrollment.apple.com" | sudo tee -a /etc/hosts
    echo "0.0.0.0 deviceenrollment.apple.com" | sudo tee -a /etc/hosts
    echo "0.0.0.0 gdmf.apple.com" | sudo tee -a /etc/hosts
    echo "0.0.0.0 acmdm.apple.com" | sudo tee -a /etc/hosts
    echo "0.0.0.0 albert.apple.com" | sudo tee -a /etc/hosts
    sudo rm /var/db/ConfigurationProfiles/Settings/.cloudConfigHasActivationRecord
    sudo rm /var/db/ConfigurationProfiles/Settings/.cloudConfigRecordFound
    sudo rm /var/db/ConfigurationProfiles/Settings/.cloudConfigTimerCheck
    sudo rm /var/db/ConfigurationProfiles/Settings/.cloudConfigurationBits
    sudo touch /var/db/ConfigurationProfiles/Settings/.cloudConfigProfileInstalled
    sudo touch /var/db/ConfigurationProfiles/Settings/.cloudConfigRecordNotFound
    yes | sudo profiles remove -all
    sudo launchctl disable system//System/Library/LaunchDaemons/com.apple.ManagedClientAgent.enrollagent
    ```
   - Especially for the `sudo rm` parts, it's best if it says the directory doesn't exist. 
   - We're adding all of these to the hosts file. From personal experience, only the `tee` option works. ALL of these servers are for Apple to check our MDM status, damn
   - macOS also uses flagging systems based on hidden files. We can use this to our advantage by creating files to represent a flag but with nothiing in it. 
   - Last 2 lines are practically useless but it's good practice to pipe yes to remove all, and disable the LaunchAgent. If a post install internet required flag is in your Mac's T2 nonvolatile secure memory (or even on the logic board), the LaunchAgent trick won't work. 
2. Restart and go to Recovery. 
3. Re-enable `csrutil` by going back to Terminal and type `csrutil enable`. 
4. Congrats, you are safe to connect to the internet! 

## (Optional): Complete bypass (Hack, not recommended)
1. This is completely overkill for average day users. You will likely never encounter a single situation that requires complete bypass. This bypass method is a bit fragile but it removes the tools to check for configuration profiles entirely. 
   - Advice: Don't do this if you don't know what a LaunchDaemon is! 
2. DO NOT DO THIS: When you type in `profiles renew -type enrollment` on an MDM enrolled Mac, even if it's bypassed, it will still show an MDM enroll notification, prompting you once every few hours to enroll. If you are sure you'll never touch this command, then don't follow this guide. 
3. Go back to Recovery and disable SIP, don't reboot yet. Enter this: `csrutil authenticated-root disable`
   - This removes the requirement inside the T2/Secure Enclave for a "Signed System Volume (SSV)". 
   - T2 accepts APFS preboot/boot snapshots only if it's signed by Apple. Otherwise, it reverts back to the "last good" snapshot (aka, the latest snapshot that's signed by Apple). 
   - So, even if we disable SIP to do some changes, especially in `/Volumes/Macintosh HD/System/Library`, it gets very angry and only chooses to boot from the last notarized snapshot, making our changes ineffective. 
   - Using authenticated-root basically tells T2 that it's safe to boot from a snapshot we modified but isn't signed by Apple, disabling SSV. 
   - NOTE: For this bypass to keep working, you WILL have to keep authenticated-root to disabled. Or else, it will always revert back to the last good Apple approved snapshot. This does NOT harm your Mac's SIP status; you can still re-enable SIP later on. It poses no harm to any viruses except yourself, since you're now telling it to boot from a seemingly custom unrestricted OS. Do at your own risk, and if you're uncomfortable, it's probably not for you. 
4. Reboot to desktop and open terminal. Run `mount | grep "(apfs, sealed, local, read-only, journaled)"`. 
5. Remember the disk location (for example, at `/dev/diskXsXsX`, replace X with numbers). 
6. Make a mount point at the disk: `mkdir ~/livemount`
7. Mount the writable system vol: `sudo mount -o nobrowse -t apfs /dev/diskXsXsX ~/livemount`
   - Replace with your own disk location! 
8. Rename all the profile configuration files to make them invalid. 
    ```zsh
    sudo mv ~/livemount/System/Library/CoreServices/ManagedClient.app ~/livemount/System/Library/CoreServices/ManagedClient.app.disabled 
    sudo mv ~/livemount/usr/libexec/cloudconfigurationd ~/livemount/usr/libexec/cloudconfigurationd.disabled
    sudo mv ~/livemount/usr/libexec/mdmclient ~/livemount/usr/libexec/mdmclient.disabled
    sudo mv /System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd.plist ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.cloudconfigurationd.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.enroll.plist ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.enroll.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.mechanism.plist ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.mechanism.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.plist ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.startup.plist ~/livemount/System/Library/LaunchDaemons/com.apple.ManagedClient.startup.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.mdmclient.daemon.plist ~/livemount/System/Library/LaunchDaemons/com.apple.mdmclient.daemon.plist.disabled
    sudo mv ~/livemount/System/Library/LaunchDaemons/com.apple.mdmclient.runatboot.plist ~/livemount/System/Library/LaunchDaemons/com.apple.mdmclient.runatboot.plist.disabled
    sudo mv ~/livemount/private/var/db/ConfigurationProfiles ~/livemount/private/var/db/ConfigurationProfiles.old
    sudo bless --mount ~/livemount --bootefi --create-snapshot
    ```
   - If you don't understand any of this here it's probably safe to NOT DO THIS! 
   - Notice here we're basically destroying all the things relevant to MDM and adding `.disabled`. While we could remove them, it's better to leave a backup if anything goes wrong. Suppose you know how to fix it back. If you don't then it's probably safe to NOT DO THIS! 
   - There is 1 app in CoreServices, 2 services in libexec, 1 folder to store profiles, and 7 LaunchDaemons. It's totally fine if some (or all) of these don't pass through the `mv`, some of this is only set up once the system knows you have a profile (but isn't installed). 
   - Lastly we bless the disk and make a snapshot out of it. This snapshot is not signed by Apple, so in order for this to boot, authenticated-root must be off. 
9. Reboot into RecoveryOS to disable SIP, but not authenticated-root if you want the changes you just made to occur. 
10. Done! Very Easy. 

> willuhd
