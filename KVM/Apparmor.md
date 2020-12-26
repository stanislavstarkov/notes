# TLDR
```bash
root@nas-1:~# cat /etc/apparmor.d/local/abstractions/libvirt-qemu
```
```
  /dev/zd* rwk,
```
###### File Permission Access Modes

File permission access modes consist of combinations of the following modes:
| attr | description |
|---|---|
| r | Read mode |
| w | Write mode (mutually exclusive to a) |
| a | Append mode (mutually exclusive to w) |
| k | File locking mode |
| l | Link mode |
| link FILE -> TARGET | Link pair rule (cannot be combined with other access modes) |
