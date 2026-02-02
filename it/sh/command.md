# shコマンド


## ps

プロセス情報を表示する。

### オプション

- `aux`: cpu, memory使用率が出る  
    - `a`: 全ユーザのプロセス表示
    - `u`: user friendlyな表示
    - `x`: 制御端末を持たないプロセス（バックグラウンド）も表示
- `ef`: プロセスの親子関係が分かりやすい  
    - `e`: 全プロセスを表示
    - `f`: full format（全項目表示）


## stat

ファイルの情報を表示する。  
lsの詳しい版。  

### オプション

- `f`: ファイルシステムの情報を表示
- `c`: 表示する情報の選択。  
    引数で対象を指定する。`f`オプションの指定有無により、内容が異なる  
    例: `%T`: ファイルシステムの種類(`f`あり)/マイナーデバイス番号(`f`なし)

### 利用例

cgroupsのバージョン確認

```
# stat -fc %T /sys/fs/cgroup
cgroup2fs
```

## nproc

CPU数を確認するコマンド。  

```
ubuntu@RYUPC:~$ nproc
2
```

## top

CPUの利用状況を確認するコマンド。  
表示中に`1`を入力することで、各CPUごとの情報を表示できる。  

## taskset

CPUを指定してコマンドを実行するコマンド。  
対象のCPU番号は、`top`コマンドや、`/sys/devices/system/cpu/cpu<n>`を確認する。  

```
taskset -c <cpu番号> <command>
```

### 利用例

```
taskset -c 0 yes >/dev/null &
ubuntu@RYUPC:~$ ps -ef | grep yes
ubuntu      3776    1212 99 23:37 pts/4    00:00:26 yes
ubuntu@RYUPC:~$ top
top - 23:38:21 up 14 min,  1 user,  load average: 0.57, 0.18, 0.08
Tasks:  73 total,   2 running,  71 sleeping,   0 stopped,   0 zombie
%Cpu0  : 44.9 us, 55.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.7 us,  0.7 sy,  0.0 ni, 98.3 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
```

## strace

実行したコマンド中に発行されたシステムコールを確認できるコマンド。  

### 利用例

`echo`コマンド発行時、`write`システムコールが発行されていることが確認できる。  

```
ubuntu@RYUPC:~$ strace echo hello
execve("/usr/bin/echo", ["echo", "hello"], 0x7ffc0022c408 /* 36 vars */) = 0
brk(NULL)                               = 0x5881df244000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffe2179a560) = -1 EINVAL (Invalid argument)
<...>
write(1, "hello\n", 6hello
)                  = 6
<...>
```

## xxd

ファイルの内容をバイナリで確認できるコマンド。  

### 利用例

```
ubuntu@RYUPC:~$ echo hello > data
ubuntu@RYUPC:~$ xxd datra
xxd: datra: No such file or directory
ubuntu@RYUPC:~$ xxd data
00000000: 6865 6c6c 6f0a                           hello.
ubuntu@RYUPC:~$ xxd -b data
00000000: 01101000 01100101 01101100 01101100 01101111 00001010  hello.
```

## lsblk

ブロックデバイス一覧を表示する。  
*ブロックデバイス（ブロック単位のRW）<>キャラクタデバイス（文字単位のRW）

基本的に、通常のdiskは`sd<a-z>`,仮想ディスクは`vd<a-z>`,nvmeは`nvme0n<1->`という感じで、**認識順に**割り振られる（つまり、起動の度に変わりうる）。  
 -> `/etc/hosts`では、デバイス名ではなく、UUIDや、Udevのpersistent device name等を用いる。  

```
ubuntu@tmp:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   40G  0 disk
├─sda1    8:1    0   39G  0 part /
├─sda14   8:14   0    4M  0 part
├─sda15   8:15   0  106M  0 part /boot/efi
└─sda16 259:0    0  913M  0 part /boot
sr0      11:0    1    4M  0 rom
```

## flock

ロック制御をおこなうためのコマンド。  
TODO: 例

## dd


ブロック単位で、ファイルのコピー・変換を行うコマンド。  

1024B=1Kのファイルを作成する例

```
ubuntu@tmp:~$ dd if=/dev/zero of=test bs=1024 count=1
1+0 records in
1+0 records out
1024 bytes (1.0 kB, 1.0 KiB) copied, 3.5968e-05 s, 28.5 MB/s
ubuntu@tmp:~$ ls -l
total 4
-rw-rw-r-- 1 ubuntu ubuntu 1024 Feb  2 15:34 test
```

サイズは、`K`, `M`でも指定できる。  
`bs`にはメモリに乗り切るサイズを指定し、メモリのサイズ以上のファイルを作成する場合は`count`を増やす。  

```
ubuntu@tmp:~$ dd if=/dev/zero of=test2 bs=1K count=1
1+0 records in
1+0 records out
1024 bytes (1.0 kB, 1.0 KiB) copied, 0.000178916 s, 5.7 MB/s
ubuntu@tmp:~$ ls -l
total 8
-rw-rw-r-- 1 ubuntu ubuntu 1024 Feb  2 15:34 test
-rw-rw-r-- 1 ubuntu ubuntu 1024 Feb  2 15:35 test2
```


## watch

引数で指定したコマンドを定期的に実行しその結果を表示するコマンド。  

```
ubuntu@tmp:~$ watch ps
Every 2.0s: ps                                                                                   tmp: Mon Feb  2 15:37:56 2026

    PID TTY          TIME CMD
   1300 pts/0    00:00:00 bash
   1942 pts/0    00:00:00 watch
   1979 pts/0    00:00:00 watch
   1980 pts/0    00:00:00 sh
   1981 pts/0    00:00:00 ps
```

