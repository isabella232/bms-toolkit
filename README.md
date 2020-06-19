# bms-toolkit

Toolkit for installing and creating an initial Oracle database on Bare Metal Solution.

## Quick Start 

* Create a Google Cloud VM to act as a [control node](/docs/toolkit-user-guide#control-node-requirements); it should be on a VPC that has SSH access to the database hosts.
* [Extract the toolkit code](/docs/toolkit-user-guide#installing-the-toolkit) on the control node.
* Create a Cloud Storage bucket to host Oracle software images.  
```bash
gsutil mb -b on gs://installation-media-1234
```
* [Download software](/docs/toolkit-user-guide#downloading-and-staging-the-oracle-software) from Oracle and populate the bucket.  Use [check-swlib.sh](/docs/toolkit-user-guide#validating-media) to determine which files are required for your Oracle version.
* Create a SSH key, and populate to an anisble user on each database host.
* Create a JSON file `db1_asm.json` with ASM disk devices
```json
[{
  "diskgroup": "DATA",
  "disks": [
    { "name": "DATA_1", "blk_device": "/dev/mapper/3600a098038314352502b4f782f446155" },
  ]},
 {
  "diskgroup": "RECO",
  "disks": [
    { "name": "RECO_1", "blk_device": "/dev/mapper/3600a098038314352502b4f782f446162" },
  ]}
]
```
* Create a JSON file `db1_mounts.json` with disk mounts
```json
[
    {
        "purpose": "software",
        "blk_device": "/dev/mapper/3600a098038314352502b4f782f446161",
        "name": "u01",
        "fstype":"xfs",
        "mount_point":"/u01",
        "mount_opts":"defaults"
    }
]
```
* Execute `install-oracle.sh`
```bash
bash install-oracle.sh \
--ora-swlib-bucket gs://installation-media-1234 \
--instance-ssh-user ansible \
--instance-ssh-key ~/.ssh/id_rsa \
--backup-dest /u01/rman \
--ora-swlib-path /u01/oracle_install \
--ora-version 19 \
--ora-swlib-type gcs \
--ora-asm-disks db1_asm.json \
--ora-data-mounts db1_mounts.json \
--ora-data-diskgroup DATA \
--ora-reco-diskgroup RECO \
--ora-db-name orcl \
--instance-ip-addr 172.16.1.1
```

Full documentaiton is available in the [user guide](/docs/toolkit-user-guide)

## Destructive cleanup

An ansible role and playbook performs a [desructive brute-force removal](/docs/toolkit-user-guide#destructive-cleanup) of Oracle software and configuration.  It does not remove other host prerequisits.

Run the desructive brute-force Oracle software removal with:

`cleanup-oracle.sh` or `ansible-playbook brute-cleanup.yml`

## Contributing to the project

Contributons and pull requests are welcome.  See docs/contributing.md and docs/code-of-conduct.md for details.

## The fine print

This product is [licensed](LICENSE.md) under the Apache 2 license.  This is not an officially supported Google project
