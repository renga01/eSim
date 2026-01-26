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
### Fix Applied:
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


##  Issue 3 – Volare Dependency Failure (xz-utils)

### Problem
During the eSim installation, the process stopped at the step:
Installing volare
E: Invalid operation xz-utils

This caused the installer to terminate and prevented further dependencies from being installed.

### Root Cause

Inside the installer script, the command used to install the dependency was syntactically incorrect:

The script contained:

`sudo apt-get xz-utils`

This is invalid because apt-get requires an operation such as install, remove, etc.
As a result, apt treated xz-utils as an invalid operation and failed.

### Fix Applied

The command was corrected to use proper apt-get syntax.

The change made:

Before:
`sudo apt-get xz-utils`

After:
`sudo apt-get install -y xz-utils`

No other logic was changed. This ensures universal compatibility across Ubuntu versions.

### Result

After applying this fix:

xz-utils installs correctly

pip3 install volare executes successfully

The eSim installer proceeds without interruption

This fix resolves a critical installer-breaking bug affecting all systems, not just Ubuntu 25.04.

---

## Issue 4 – KiCad Dependency Failure (libgit2-1.8) on Ubuntu 25.04
Problem

During installation, the script failed while installing KiCad with the following error:

kicad : Depends: libgit2-1.8 (>= 1.8.0) but it is not installable
E: Unable to correct problems, you have held broken packages.

This stopped the entire installation process and prevented testing of further components.

# Root Cause

The KiCad packages available for Ubuntu 25.04 currently depend on:

libgit2-1.8

However, this library is not available in Ubuntu 25.04 repositories, making the dependency impossible to satisfy.
This is an upstream packaging issue and cannot be resolved directly from the installer script.

Action Taken (Workaround Applied)

As permitted by the task instructions (“you may comment out some part of the script so that you may move on to the next bug”), I temporarily skipped KiCad installation to allow the rest of the installer to continue.

The following lines were commented in the installer flow:
``` bash
# installKicad
# copyKicadLibrary
```

And replaced with:
``` bash
echo "Skipping KiCad installation on Ubuntu 25.04 due to unresolved dependency (libgit2-1.8)"
```
## Result

The installer no longer stops due to KiCad dependency failure

Remaining components of eSim continue to install correctly

This allowed further testing and debugging of the installer

The issue is clearly documented for future maintainers
## Summary of Improvements

- Added compatibility for Ubuntu 25.04 in the main installer  
- Fixed incorrect fallback behavior for KiCad installation  
- Prevented apt failures caused by unsupported PPAs  
- Improved robustness of the installer for newer Ubuntu versions  

---


## Conclusion

This task provided real-world exposure to open-source debugging and maintenance.  
By identifying compatibility issues and modifying the installer scripts, I improved eSim’s support for Ubuntu 25.04 and documented the entire process for future users.

