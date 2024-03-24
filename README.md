# GCB-Ansible

GCB-Ansible-2.0是依據「政府組態基準(GCB) - 國家資通安全研究院」 作業系統說明文件： TWGCB-01-008_Red Hat Enterprise Linux 8政府組態基準說明文件v1.0_1100924.pdf 撰寫 。

## 執行環境

* OS: Debian11
* ansible version: ansible 2.10.8

## 部屬環境

* OS: Oracle-Linux-9, CentOS-7

## 部屬前設定

### 目標主機設定

### RedHat6

1. 掛載光碟
2. 安裝必要套件

```bash
yum install -y libselinux-python
```

### 開發環境設定

```bash
tee /etc/sudoers.d/pollo-develop<<EOF
ansible ALL=(ALL)   NOPASSWD: ALL
EOF
```

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

## 依需求設定



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

### 全系統GPG簽章驗證

依據下列規範檢查，如果大於零需要寫特殊原因

* TWGCB-01-003-0009
* TWGCB-01-003-0010

```bash
grep gpgcheck=0 /etc/yum.conf /etc/yum.repos.d/* | wc -l
```

### 使用RPM驗證套件完整性

依據下列規範檢查，如果有列出需要特別注意

* TWGCB-01-003-0012

```bash
rpm -qVa | awk '$2!="c" {print $0}'
```

### 手

## 注意事項

* 設定完 playbook-base.yml 需要重新開機