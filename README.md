# GCB-Ansible

GCB-Ansible-2.0是依據「政府組態基準(GCB) - 國家資通安全研究院」 作業系統說明文件： TWGCB-01-008_Red Hat Enterprise Linux 8政府組態基準說明文件v1.0_1100924.pdf 撰寫 。

## 執行環境

* OS: Debian11
* ansible version: ansible 2.10.8

## 部屬環境

* OS: Oracle-Linux-9

## 部屬前設定

### (Optional)建立帳號:ansible

```bash
sudo groupadd ansible
sudo useradd -g ansible -d /home/ansible -s /bin/bash ansible
sudo usermod -aG wheel ansible
sudo passwd ansible
```

### ssh-keygen

1. 新增存放目的資料夾

```bash
mkdir -p /home/jameschang/Documents/gitContent/jameschang/GCB-Ansible/.ssh
```

2. 產生ssh key，並放置在專案目錄中

```bash
ssh-keygen # /home/jameschang/Documents/gitContent/jameschang/GCB-Ansible/.ssh/id_rsa
```

3. 將公鑰設定放到伺服器上

```bash
ssh-copy-id -i .ssh/id_rsa ansible@10.192.3.100
```

4. 連接測試

```bash
ssh -i .ssh/id_rsa ansible@10.192.3.100
```

### log 設定

```bash
mkdir -p logs
touch logs/ansible.log
```

## tools

### 檢查部屬OS版本資訊

```bash
ansible-playbook check-os-version.yml -i inventorys/develop
```

## 安裝 OS 時設定

* 獨立磁區(安裝時處理)
  * TWGCB-01-008-0008: /var
  * TWGCB-01-008-0009: /var/tmp
  * TWGCB-01-008-0013: /var/log
  * TWGCB-01-008-0014: /var/log/audit
  * TWGCB-01-008-0015: /home

## 手動設定項目

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

#### TWGCB-01-008-0004 TWGCB-01-008-0007: 設定/tmp

* TWGCB-01-008-0004: 設定/tmp目錄之檔案系統
* TWGCB-01-008-0005: 設定/tmp目錄之nodev選項
* TWGCB-01-008-0006: 設定/tmp目錄之nosuid選項
* TWGCB-01-008-0007: 設定/tmp目錄之noexec選項

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

* TWGCB-01-008-0011: 設定/var/tmp目錄之nosuid選項
* TWGCB-01-008-0012: 設定/var/tmp目錄之noexec選項

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

* TWGCB-01-012-0016: 設定/home目錄之nodev選項

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

* TWGCB-01-012-0017: 設定/dev/shm目錄之nodev選項
* TWGCB-01-012-0018: 設定/dev/shm目錄之nosuid選項
* TWGCB-01-012-0019: 設定/dev/shm目錄之noexec選項

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

* TWGCB-01-008-0020: 設定可攜式儲存裝置之nodev選項
* TWGCB-01-008-0021: 設定可攜式儲存裝置之nosuid選項
* TWGCB-01-008-0022: 設定可攜式儲存裝置之noexec選項

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

## 依需求設定

### TWGCB-01-008-0266

啟用限制存取SSH功能，有助於確保只有授權使用者才能透過SSH遠端存取系統

* `/etc/ssh/sshd_config`檔案可設定以下參數，以限制可透過SSH存取系統的使用者與群組：

1. AllowUsers：由空格分隔的使用者名稱組成，設定允許登入的使用者
2. AllowGroups：由空格分隔的群組名稱組成，設定允許登入的群組
3. DenyUsers：由空格分隔的使用者名稱組成，設定拒絕登入的使用者
4. DenyGroups：由空格分隔的群組名稱組成，設定拒絕登入的群組

設定完成後執行以下指令，重新啟動SSH服務使其生效

```bash
systemctl restart sshd
```

### TWGCB-01-008-0093: chrony校時設定

### TWGCB-01-008-0096: SNMP服務

```bash
dnf install net-snmp net-snmp-utils net-snmp-libs net-snmp-devel

systemctl stop snmpd

net-snmp-create-v3-user -ro -A snmp@PWD1 -a SHA -X snmp@PWD1 -x AES snmpAdmin
```

* /etc/snmp/snmpd.conf

將包含com2sec、group、view及access參數之行內容註解(新增#符號於行首)，以停用SNMPv1與SNMPv2，範例如下：

```conf
#com2sec notConfigUser  default       public
#group   notConfigGroup v1           notConfigUser
#group   notConfigGroup v2c          notConfigUser
#view    systemview    included   .1.3.6.1.2.1.1
#view    systemview    included   .1.3.6.1.2.1.25.1.1
#access  notConfigGroup  ""      any       noauth    exact  systemview none none
```

### TWGCB-01-008-0137,TWGCB-01-008-0138: 稽核日誌檔案所有權

1.  開啟終端機，執行以下指令，尋找稽核日誌檔案

```bash
grep -iw log_file /etc/audit/auditd.conf
```

2. 執行以下指令，設定稽核日誌檔案擁有者與群組, 設定稽核日誌檔案權限為600或更低權限

```bash
chown root:root (稽核日誌檔案名稱)
chmod 600 (稽核日誌檔案名稱)
```

### TWGCB-01-008-0205,TWGCB-01-008-0206: at.allow與cron.allow檔案

### 帳戶設定

* TWGCB-01-008-0225
* TWGCB-01-008-0226
* TWGCB-01-008-0227
* TWGCB-01-008-0228

### 安裝 GUI 設定

* TWGCB-01-012-0234
* TWGCB-01-008-0235
* TWGCB-01-008-0236
* TWGCB-01-008-0239

### 磁碟檢查項目

* TWGCB-01-008-0061
* TWGCB-01-008-0062
* TWGCB-01-008-0063
* TWGCB-01-008-0064
* TWGCB-01-008-0065

```bash
find (partition) -xdev -type f -perm -0002
find (partition) -xdev -nouser
find (partition) -xdev -nogroup
find (partition) -xdev -type d -perm -0002 -uid +999 -print
```

## TWGCB-01-008-0134,TWGCB-01-008-0135,TWGCB-01-008-0186

* /etc/default/grub

1. 新增「,audit=1, audit_backlog_limit=8192」
2. 移除所有「selinux=0」與「enforcing=0」內容

```bash
GRUB_CMDLINE_LINUX="audit=1 audit_backlog_limit=8192"
```

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 注意事項

* 設定完 playbook-base.yml 需要重新開機