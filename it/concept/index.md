# 概念系

## procfs

プロセス情報のLinux上の表現。ファイルシステム上で表現しているため、fs。  
`/proc/`にマウントされている。これはメモリ上の情報である。

`procps`パッケージはそのutil的なもので、`ps`コマンドなどが含まれる。  

```
ubuntu@tmp:~$ which ps
/usr/bin/ps
ubuntu@tmp:~$ dpkg -S /usr/bin/ps
procps: /usr/bin/ps
ubuntu@tmp:~$ dpkg -L procps | grep bin
/usr/bin
/usr/bin/free
/usr/bin/kill
/usr/bin/pgrep
/usr/bin/pidwait
/usr/bin/pmap
/usr/bin/ps
/usr/bin/pwdx
/usr/bin/skill
/usr/bin/slabtop
/usr/bin/tload
/usr/bin/top
/usr/bin/uptime
/usr/bin/vmstat
/usr/bin/w
/usr/bin/watch
/usr/sbin
/usr/sbin/sysctl
/usr/bin/pkill
/usr/bin/snic
```

## キャッシュ  

CPUのキャッシュ。CPUとメインメモリの速度差を補う。  
キャッシュに書き込んだ内容をメインメモリに書き出すタイミングとして２通りある。  

- ライトバック: バックグラウンドで同期  
    速度優先。ただし実装は複雑。
- ライトスルー: キャッシュへの書き込み時、常にメインメモリにも書き出す  
    整合性優先。ただし書き込みの時はキャッシュの恩恵が薄い。

`/sys/devices/system/cpu/cpu{n}/cache`以下がCPUごとのキャッシュ情報。  

### 例

`type`は、`Data`/`Instruction`の２つ。  
例えば、L1キャッシュは`index0`, `index1`、L2キャッシュは`index2`で構成される、という感じ。  

```
ubuntu@tmp:~$ cat /sys/devices/system/cpu/cpu0/cache/index0/type
Data
ubuntu@tmp:~$ cat /sys/devices/system/cpu/cpu0/cache/index1/type
Instruction
ubuntu@tmp:~$ cat /sys/devices/system/cpu/cpu0/cache/index2/type
Unified
```

## tmpfs/brd

メモリ上のファイルシステム。  
`tmpfs`はブロックデバイス自体存在せず（`nodev`必須）、`brd`はブロックデバイスを仮想的に作成する。  
動作上の大きな違いとしては、`umount`した場合、`tmpfs`では即座にデータが消えるが、`brd`の場合はデバイス自体を削除するまでは再mountすれば残っている。  
rebootした場合はいずれも消える（結局メモリ上のデータなので）

### 例

`tmpfs`

```
ubuntu@tmp:~$ mkdir mnt
ubuntu@tmp:~$ sudo mount -t tmpfs nodev mnt
ubuntu@tmp:~$ touch mnt/aaa
ubuntu@tmp:~$ ls mnt/
aaa
```

`brd`

```
ubuntu@tmp:~$ modeprobe brd
ubuntu@tmp:~$ ls /dev/ram*
/dev/ram0  /dev/ram10  /dev/ram12  /dev/ram14  /dev/ram2  /dev/ram4  /dev/ram6  /dev/ram8
/dev/ram1  /dev/ram11  /dev/ram13  /dev/ram15  /dev/ram3  /dev/ram5  /dev/ram7  /dev/ram9
ubuntu@tmp:~$ sudo parted /dev/ram0 p
Error: /dev/ram0: unrecognised disk label
Model: RAM Drive (brd)
Disk /dev/ram0: 67.1MB
Sector size (logical/physical): 512B/4096B
Partition Table: unknown
Disk Flags:

ubuntu@tmp:~$ sudo mkfs.ext4 /dev/ram0
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 16384 4k blocks and 16384 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

ubuntu@tmp:~$ sudo mount /dev/ram0 mnt/
ubuntu@tmp:~$ mount | grep ext4
/dev/sda1 on / type ext4 (rw,relatime,discard,errors=remount-ro,commit=30)
/dev/sda16 on /boot type ext4 (rw,relatime)
/dev/ram0 on /home/ubuntu/mnt type ext4 (rw,relatime)

ubuntu@tmp:~$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
tmpfs             201536    1052    200484   1% /run
/dev/sda1       39535100 1712464  37806252   5% /
tmpfs            1007664       0   1007664   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
/dev/sda16        901520   62456    775936   8% /boot
/dev/sda15        106832    6250    100582   6% /boot/efi
tmpfs             201532      12    201520   1% /run/user/1000
/dev/ram0          57300      24     52692   1% /home/ubuntu/mnt ★ブロックデバイスなのでdf一覧に出る

ubuntu@tmp:~$ sudo umount mnt
ubuntu@tmp:~$ sudo modprobe -r brd
ubuntu@tmp:~$ ls /dev/ram*
ls: cannot access '/dev/ram*': No such file or directory
```

## dirty

DBや、キャッシュの話で出てくる`dirty`とは、メモリ上のデータとディスク上のデータが異なるデータのこと。  
未確定（不完全）なデータを「汚れている」と表現している。  