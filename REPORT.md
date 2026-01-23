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

###Fix Applied:
Modified run_version_script() to treat Ubuntu 25.04 same as 24.04:

"25.04")
    SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
    ;;

       exit 1

