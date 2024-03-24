# 日誌與稽核

##

### TWGCB-01-003-0154: 記錄特權指令使用情形

1. 執行下列指令，針對每個磁區(PART)的特權指令產生稽核規則：

```bash
find PART -xdev \( -perm -4000 -o -perm -2000 \) -type f | awk '{print \ "-a always,exit -F path=" $1 " -F perm=x -F auid>=500 -F auid!=4294967295 \ -k privileged" }'
```

2. 將上述指令所產出之規則內容新增至/etc/audit/audit.rules檔案中

### TWGCB-01-003-0155: 記錄資料匯出至媒體


1. 先使用uname -m，查詢作業系統是32位元(b32)或64位元(b64)
2. 編輯/etc/audit/audit.rules檔案，新增或修改成以下內容(32位元系統請將ARCH改為b32，64位元系統請將ARCH改為b64)：

```bash
-a always,exit -F arch=ARCH -S mount -F auid>=500 -F auid!=4294967295 -k export
```

### TWGCB-01-003-0156: 記錄檔案刪除事件

1. 先使用uname -m，查詢作業系統是32位元(b32)或64位元(b64)
2. 編輯/etc/audit/audit.rules檔案，新增或修改成以下內容(32位元系統請將ARCH改為b32，64位元系統請將ARCH改為b64)：

```bash
-a always,exit -F arch=ARCH -S unlink -S unlinkat -S rename -S renameat -F auid>=500 -F auid!=4294967295 -k delete
```

### TWGCB-01-003-0158: 記錄核心模組掛載與卸載事件

1. 先使用uname -m，查詢作業系統是32位元(b32)或64位元(b64)
2. 編輯/etc/audit/audit.rules檔案，新增或修改成以下內容(32位元系統請將ARCH改為b32，64位元系統請將ARCH改為b64)：

```bash

-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=ARCH -S init_module -S delete_module -k modules
```
