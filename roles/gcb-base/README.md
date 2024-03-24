# 基礎服務

* version: 1.0

## 手動處理項目

### 獨立磁區

下列目錄需要獨立磁區處理，建議在安裝作業系統時處理

* /var : 依下列 TWGCB-ID 要求
  * TWGCB-01-003-0002
  * TWGCB-01-008-0008
* /var/tmp : 依下列 TWGCB-ID 要求
  * TWGCB-01-003-0001
  * TWGCB-01-008-0009
* /var/log : 依下列 TWGCB-ID 要求
  * TWGCB-01-003-0003
  * TWGCB-01-008-0013
* /var/log/audit : 依下列 TWGCB-ID 要求
  * TWGCB-01-003-0004
  * TWGCB-01-008-0014
* /home : 依下列 TWGCB-ID 要求
  * TWGCB-01-003-0005
  * TWGCB-01-008-0015

## 手動設定項目

### 裝置檢查

```bash
#!/bin/bash
# TWGCB-01-003-0043
find PART -xdev -type d \( -perm -0002 -a ! -perm -1000 \) -print
# TWGCB-01-003-0044
find PART -xdev -type f -perm -0002 -print
# TWGCB-01-003-0045
find PART -xdev -type f -perm -2000 -print
# TWGCB-01-003-0046
find PART -xdev -type f -perm -4000 -print
# TWGCB-01-003-0047
find PART -xdev \( -nouser -o -nogroup \) -print
# TWGCB-01-003-0048
find PART -xdev \( -nouser -o -nogroup \) -print
# TWGCB-01-003-0049
find PART -xdev -type d -perm -0002 -uid +500 -print

grep -E "ttyS0|ttyS1" /etc/securetty
```

### 基本項目

以下是通用項目

* /etc/fstab

```bash
tmpfs /tmp tmpfs defaults,rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs defaults,nodev,nosuid,noexec 0 0
```

```bash
systemctl unmask tmp.mount
systemctl enable tmp.mount
mount -o remount,nodev,nosuid,noexec /dev/shm
```

* /etc/systemd/system/local-fs.target.wants/tmp.mount

```conf
[Mount]
What=tmpfs
Where=/tmp
Type=tmpfs
Options=mode=1777,strictatime,noexec,nodev,nosuid
```

#### TWGCB-01-003-0013: 非root分割區啟用nodev選項

編輯/etc/fstab檔案，在掛載點不是「/」且檔案系統為ext2或ext3之列，於第4欄加入「,nodev」

#### TWGCB-01-003-0014 TWGCB-01-003-0016: 可攜式儲存裝置

* 可攜式儲存裝置啟用nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0014
* 可攜式儲存裝置啟用noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0015
* 可攜式儲存裝置啟用nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0016

#### TWGCB-01-008-0004 TWGCB-01-008-0007: 設定/dev/shm

* 可攜式儲存裝置啟用nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0020
* 可攜式儲存裝置啟用noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0022
* 可攜式儲存裝置啟用nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0021

#### TWGCB-01-003-0023: 設定/var/tmp

* 將/var/tmp綁定掛載到/tmp，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0023

##### 說明

▪  這項原則設定決定是否將/var/tmp系統服務暫存資料目錄，綁定掛載到/tmp
▪  啟用這項原則，可將/var/tmp與/tmp暫存資料皆寫入/tmp目錄中，讓/var/tmp與/tmp擁有相同保護機制
▪  若使用者欲透過/var/tmp中之暫存檔案執行惡意程式，將會受限於/tmp權限限制，防止惡意程式執行

##### 設定方法

編輯/etc/fstab，新增或修改成以下內容：

```
/tmp /var/tmp none rw,noexec,nosuid,nod ev,bind 0 0
```

#### TWGCB-01-008-0004 TWGCB-01-008-0007: 設定/tmp

* 設定/tmp目錄之檔案系統，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0004
* 設定/tmp目錄之nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0017
  * TWGCB-01-008-0005
* 設定/tmp目錄之nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0018
  * TWGCB-01-008-0006
* 設定/tmp目錄之noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-003-0019
  * TWGCB-01-008-0007

##### 說明

###### TWGCB-01-008-0004: 設定/tmp目錄之檔案系統

* 這項原則設定決定/tmp目錄是否使用tmpfs檔案系統
* /tmp是具有全域寫入權限之目錄，所有使用者與某些應用程式都將其用於暫存檔案
* tmpfs是一個不存在於實體硬碟上，而是駐存在記憶體的特殊檔案系統，可提供優於傳統機械硬碟的存取速度
* 將/tmp目錄掛載到tmpfs，可於掛載選項上使用noexec選項，指定/tmp目錄不能啟動可執行二進制檔案，使攻擊者不能安裝執行惡意程式以降低風險

###### TWGCB-01-008-0005: 設定/tmp目錄之nodev選項

* 這項原則設定決定/tmp目錄是否啟用nodev選項，以禁止在/tmp目錄中建立裝置檔案
* 由於/tmp目錄用途不在於支援裝置，設定nodev選項以確保使用者無法在/tmp目錄建立裝置檔案或存取隨機硬體裝置，降低惡意程式感染風險
* 可設定之參數如下：
  1. dev：允許建立裝置檔案
  2. nodev：禁止建立裝置檔案

###### TWGCB-01-008-0006: 設定/tmp目錄之nosuid選項

* 這項原則設定決定/tmp目錄是否啟用nosuid選項，以禁止/tmp目錄存在具有SUID屬性的檔案
* SUID(Set User ID)是針對可執行二進制檔案(Binary file)設計的一項功能，任何使用者執行該程式時，會以該程式之擁有者的身分執行，藉由短暫提權讓一般使用者存取未被授權之檔案
* 可設定之參數如下：
  1. suid：允許存在具有SUID屬性的檔案
  2. nosuid：禁止存在具有SUID屬性的檔案

###### TWGCB-01-008-0007: 設定/tmp目錄之noexec選項

* 這項原則設定決定/tmp目錄是否啟用noexec選項，以禁止在/tmp目錄中啟動可執行二進制檔案
* /tmp目錄主要做為暫時存放檔案之用，禁止使用者啟動可執行二進制檔案，以避免感染惡意程式
* 可設定之參數如下：
  1. exec：允許啟動可執行二進制檔案
  2. noexec：禁止啟動可執行二進制檔案

##### 設定方法

執行以下任一操作以設定/tmp目錄之檔案系統：

1. 開啟終端機，執行下列指令，編輯/etc/fstab

```bash
vim /etc/fstab
```

/tmp的掛載設定範例如下：

```conf
# TWGCB-01-008-0004
tmpfs /tmp tmpfs defaults,rw,nosuid,nodev,noexec,relatime 0 0
```

2. 開啟終端機，執行下列指令，運用systemd掛載/tmp：

```bash
systemctl unmask tmp.mount
systemctl enable tmp.mount
```

編輯`/etc/systemd/system/local-fs.target.wants/tmp.mount`以掛載/tmp：

```bash
sudo mkdir -p /etc/systemd/system/local-fs.target.wants/
vim /etc/systemd/system/local-fs.target.wants/tmp.mount
```

```conf
[Mount]
What=tmpfs
Where=/tmp
Type=tmpfs
Options=mode=1777,strictatime,noexec,nodev,nosuid
```

#### TWGCB-01-008-0011 TWGCB-01-008-0012: 設定/var/tmp

* 設定/var/tmp目錄之nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0011
* 設定/var/tmp目錄之noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0012

##### 說明

###### TWGCB-01-008-0011: 設定/var/tmp目錄之nosuid選項

▪  這項原則設定決定/var/tmp目錄是否啟用nosuid選項，以禁止/var/tmp目錄存在具有SUID屬性的檔案
▪  SUID(Set User ID)是針對可執行二進制檔案(Binary file)設計的一項功能，任何使用者執行該程式時，會以該程式之擁有者的身分執行，藉由短暫提權讓一般使用者存取未被授權之檔案
▪  可設定之參數如下：
(1) suid：允許存在具有SUID屬性的檔案
(2) nosuid：禁止存在具有SUID屬性的檔案

###### TWGCB-01-008-0012: 設定/var/tmp目錄之noexec選項

▪  這項原則設定決定/var/tmp目錄是否啟用noexec選項，以禁止在/var/tmp目錄中啟動可執行二進制檔案
▪  /var/tmp目錄主要做為暫時存放檔案之用，禁止使用者啟動可執行二進制檔案，以避免感染惡意程式
▪  可設定之參數如下：
(1) exec：允許啟動可執行二進制檔案
(2) noexec：禁止啟動可執行二進制檔案

##### 設定方法

* 編輯`/etc/fstab`檔案，在掛載點為/var/tmp列，於第4欄加入「,noexec,nosuid」

```conf
# TWGCB-01-008-0012
/dev/mapper/ol-var_tmp  /var/tmp                xfs     defaults,noexec,nosuid        0 0
```

* 開啟終端機，執行下列指令，重新掛載/var/tmp：

```bash
systemctl daemon-reload 
mount -o remount,noexec,nosuid /var/tmp
```

#### TWGCB-01-012-0016: 設定/home目錄

##### 說明

* 設定/home目錄之nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-012-0016

###### TWGCB-01-012-0016: 設定/home目錄之nodev選項

* 這項原則設定決定/home目錄是否啟用nodev選項，以禁止在/home目錄中建立裝置檔案
* /home目錄是系統預設的使用者家目錄(Home directory)。每當新增1個一般使用者帳號時，預設的使用者家目錄都會建立在/home目錄下
* 由於/home目錄用途不在於支援裝置，設定nodev選項以確保使用者無法在/home目錄建立裝置檔案或存取隨機硬體裝置，降低惡意程式感染風險
* 可設定之參數如下：
1. dev：允許建立裝置檔案
2. nodev：禁止建立裝置檔案

##### 設定方法

* 編輯`/etc/fstab`檔案，在掛載點為/home列，於第4欄加入「,nodev」

```conf
/dev/mapper/ol-home     /home                   xfs     defaults,nodev        0 0
```

* 開啟終端機，執行下列指令，重新掛載/home：

```bash
systemctl daemon-reload
mount -o remount,nodev /home
```

#### TWGCB-01-012-0017 TWGCB-01-012-0019: 設定/dev/shm目錄

* 設定/dev/shm目錄之nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-012-0017
* 設定/dev/shm目錄之nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-012-0018
* 設定/dev/shm目錄之noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-012-0019

##### 說明

###### TWGCB-01-012-0017: 設定/dev/shm目錄之nodev選項

* 這項原則設定決定/dev/shm目錄是否啟用nodev選項，以禁止在/dev/shm目錄中建立裝置檔案
* /dev/shm目錄是系統利用記憶體建立的虛擬磁碟空間，用以存放暫存檔案，一方面存取速度比硬碟快，另一方面可依實際檔案大小動態分配空間
* 由於/dev/shm目錄用途不在於支援裝置，設定nodev選項以確保使用者無法在/dev/shm目錄建立裝置檔案或存取隨機硬體裝置，降低惡意程式感染風險
* 可設定之參數如下：
  1. dev：允許建立裝置檔案
  2. nodev：禁止建立裝置檔案

###### TWGCB-01-012-0018: 設定/dev/shm目錄之nosuid選項

* 這項原則設定決定/dev/shm目錄是否啟用nosuid選項，以禁止/dev/shm目錄存在具有SUID屬性的檔案
* /dev/shm目錄是系統利用記憶體建立的虛擬磁碟空間，用以存放暫存檔案，一方面存取速度比硬碟快，另一方面可依實際檔案大小動態分配空間
* SUID(Set User ID)是針對可執行二進制檔案(Binary file)設計的一項功能，任何使用者執行該程式時，會以該程式之擁有者的身分執行，藉由短暫提權讓一般使用者存取未被授權之檔案
* 可設定之參數如下：
  1. suid：允許存在具有SUID屬性的檔案
  2. nosuid：禁止存在具有SUID屬性的檔案

###### TWGCB-01-012-0019: 設定/dev/shm目錄之noexec選項

* 這項原則設定決定/dev/shm目錄是否啟用noexec選項，以禁止在/dev/shm目錄中啟動可執行二進制檔案
* /dev/shm目錄是系統利用記憶體建立的虛擬磁碟空間，用以存放暫存檔案，一方面存取速度比硬碟快，另一方面可依實際檔案大小動態分配空間
* 設定noexec選項，禁止使用者啟動可執行二進制檔案，以避免感染惡意程式
* 可設定之參數如下：
  1. exec：允許啟動可執行二進制檔案
  2. noexec：禁止啟動可執行二進制檔案

##### 設定方法

* 編輯/etc/fstab檔案，在掛載點為/dev/shm列，於第4欄加入「,nodev」

```conf
# TWGCB-01-012-0019
tmpfs /dev/shm tmpfs defaults,nodev,nosuid,noexec 0 0
```

```bash
systemctl daemon-reload
mount -o remount,nodev,nosuid,noexec /dev/shm
```

#### TWGCB-01-008-0020 TWGCB-01-008-0022: 設定可攜式儲存裝置

如果有插入可攜式儲存裝置，請設定下列項目

* 設定可攜式儲存裝置之nodev選項，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0020
* 設定可攜式儲存裝置之nosuid選項，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0021
* 設定可攜式儲存裝置之noexec選項，依下列 TWGCB-ID 要求
  * TWGCB-01-008-0022

### TWGCB-01-008-0034,TWGCB-01-008-0035: /etc/sudoers.d/

* TWGCB-01-008-0034: 設定sudo指令使用pty
* TWGCB-01-008-0035: sudo自定義日誌檔案

* /etc/sudoers.d/gcb

```bash
visudo -f /etc/sudoers.d/gcb
```

```conf
Defaults use_pty
Defaults logfile="/var/log/sudo.log"
```