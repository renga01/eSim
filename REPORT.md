# eSim 2.5 Installation Issues on Ubuntu 25.04

## System Details
- OS: Ubuntu 25.04 (VirtualBox VM)
- Host OS: Ubuntu 20.04
- eSim Version: 2.5
- Installation Method: install-eSim.sh script

---

## Issue 1: Unsupported Ubuntu Version Error

### Error Observed
While running:
./install-eSim.sh --install

The script failed with:
Unsupported Ubuntu version: 25.04

### Cause of the Issue
In install-eSim.sh, the script only supports:
- 22.04
- 23.04
- 24.04

The case statement rejects any other version.

Code responsible:
```bash
case $VERSION_ID in
   "22.04") ...
   "23.04") ...
   "24.04") ...
   *)
       echo "Unsupported Ubuntu version"
```
###Fix Applied:
Modified run_version_script() to treat Ubuntu 25.04 same as 24.04:

"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
    ;;

       exit 1
       
##  Issue 2 – KiCad PPA Failure on Ubuntu 25.04

### Problem
During installation, `apt update` failed with the error:
E: The repository 'https://ppa.launchpadcontent.net/kicad/kicad-6.0-releases/ubuntu
 plucky Release' does not have a Release file.
This prevented further package installation.

### Root Cause
The script logic in `install-eSim-24.04.sh` caused Ubuntu 25.04 to fall back to using:
- ppa:kicad/kicad-6.0-releases

This PPA does not support Ubuntu 25.04, causing apt to fail.

### Fix Applied
The `installKicad` function was updated so that:
- Ubuntu 24.04 uses KiCad 8 PPA  
- Ubuntu 25.04 also uses KiCad 8 PPA  
- Only older Ubuntu versions use KiCad 6 PPA  

Additionally, the already-added broken PPA was removed manually using:
sudo add-apt-repository --remove ppa:kicad/kicad-6.0-releases


### Result
After this fix:
- apt update no longer failed  
- The installer no longer attempted to use unsupported repositories  
- Installation proceeded further correctly  

---

## Summary of Improvements

- Added compatibility for Ubuntu 25.04 in the main installer  
- Fixed incorrect fallback behavior for KiCad installation  
- Prevented apt failures caused by unsupported PPAs  
- Improved robustness of the installer for newer Ubuntu versions  

---

## Learning Outcomes

This task helped me gain practical experience with:
- Debugging Bash installation scripts  
- Understanding Linux dependency and repository issues  
- Working with PPAs and apt package management  
- Using Git branches and commits for contributions  
- Writing clear technical documentation  

---

## Conclusion

This task provided real-world exposure to open-source debugging and maintenance.  
By identifying compatibility issues and modifying the installer scripts, I improved eSim’s support for Ubuntu 25.04 and documented the entire process for future users.

