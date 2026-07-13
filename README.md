# Datimo IT Solutions - VMware NFS Import and Migration Tool

This repository contains the helper script [pve_nfs_vmware_import.py](pve_nfs_vmware_import.py) to import VMware virtual machines whose VMDK files are stored on an NFS share into Proxmox VE. The workflow is designed for a migration path where the VM is first brought up from the NFS-backed disks in Proxmox, then the disks can be moved to the final target storage.

## What this script does

- Scans VMware `.vmx` files from an NFS-mounted datastore
- Parses the VM configuration and virtual disks
- Creates Proxmox VM definitions using local VMDK descriptors that point back to the NFS-backed flat files
- Starts the VM in Proxmox
- Allows disk-by-disk migration from the temporary NFS-backed location to the final Proxmox storage

## Requirements

The script is intended to run on a Proxmox host and requires:

- Root privileges
- Proxmox commands available on the host:
  - `qm`
  - `pvesm`
- The local Storage of Proxmox must be set to support Type `disk-image`
- Python 3 with the following packages:
  - `rich`
  - `questionary`
  - `pydantic`

## Installation

On the Proxmox host, install the required Python environment first:

```bash
apt install python3.13-venv
```

Create a working directory and install the Python dependencies:

```bash
mkdir -p /opt/pve-nfs-import
cd /opt/pve-nfs-import

python3 -m venv .venv
source .venv/bin/activate

pip install rich questionary pydantic
```

Copy the migration script into the folder and run it from there:

```bash
cd /opt/pve-nfs-import
source .venv/bin/activate
python pve_nfs_vmware_import.py
```

## How to use the script

Start the script with:

```bash
python pve_nfs_vmware_import.py [--dry-run] [--nfs-root /mnt/vmware-nfs] [--yes]
```

Useful options:

- `--dry-run` - performs planning and logging without changing Proxmox state
- `--nfs-root /path/to/nfs` - points the script at the mounted VMware NFS share
- `--state-dir /path` - overrides the state directory
- `--log-dir /path` - overrides the log directory
- `--descriptor-base-dir /path` - overrides the local descriptor storage location
- `--yes` - automatically confirms prompts where possible
- `--offline-test` - runs in a test mode without executing Proxmox commands

## Migration procedure

Follow this sequence carefully:

1. Prepare the VMware VM with virtio drivers and a dummy disk
   - Ensure the guest is prepared for a Proxmox-style import path
   - Add or verify the required virtio drivers inside the guest OS
   - Create a dummy disk if required by your migration workflow

2. Uninstall VMware guest tools
   - Remove the VMware Tools package from the source guest before the import path is finalized
   - This reduces the chance of driver and service conflicts during the first boot in Proxmox

3. Start the migration with the script
   - Mount the VMware NFS export on the Proxmox host
   - Run the import script and point it to the NFS root path
   - Select the VMware VM, target storage, and network mapping when prompted

4. Shut down the VMware VM
   - Confirm that the original VMware guest is fully shut down before proceeding
   - The script will prompt you to confirm this step before it allows the Proxmox boot phase

5. Boot up on Proxmox (via the script)
   - After the import configuration is created, the script can start the Proxmox VM from the NFS-backed disks
   - Validate that the guest boots successfully and that the OS is stable

6. Migrate to the final storage
   - Once the VM is running on Proxmox and the system has been validated, migrate the disks to the final target storage using the script's disk migration step
   - This moves the imported disks from the temporary NFS-backed representation to the intended Proxmox storage backend

## Need Help
![Datimo IT Solution logo](./datimo-logo.png)  
If you need assistance with the migration process, the import script, or the Proxmox setup, please visit our website: [Datimo IT Solution](https://www.datimo.ch) or contact the Datimo IT Solution team / the responsible administrator for this environment.

## Notes

- The tool is intended for experienced administrators working with Proxmox VE and VMware migrations
- Always test with a non-production VM first
- Review the generated logs under the configured log directory if something fails
