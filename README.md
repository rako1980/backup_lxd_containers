# backup_lxd_containers
An ansible playbook to backup LXD containers. Backup as an image (not .tar.gz)

##Run as:
```
ansible-playbook backup_lxd_containers.yml -e lxd_host=lxd_hostname -e interval=daily
```
interval: daily|weekely - Creates a backup image with daily or weekly as name

##Steps:
- Get all containers from the lxd host
- Add containers to the in memory inventory group
- Then iterate through in memory inventory group:
    - remove any exisiting snapshot with named snap-backup
    - take snapshot snap-backup
    - create image from snapshot
    - remove snapshot
    - delete 3 days old image if interval=days, or 3 weeks old snapshot if interval is week

It uses the Ansible uri module connecting to local socket. On each api call where the requests are async, it waits for the operation to complete before moving to next task. Also, the playbook is executed for each container serially (serial: 1) to avoid the disk space and IO issue diring snapshotting and image creation. The best practice is to separate a filesystem for the image location (/var/lib/lxd/image), as well as use lvm, btrfs, zfs etc for the storage pool, otherwise the snapshooting is inefficient with directory based storage pool.
