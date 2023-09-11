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
groupadd ansible
useradd -g ansible -d /home/ansible -s /bin/bash ansible
usermod -aG wheel ansible
passwd ansible
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