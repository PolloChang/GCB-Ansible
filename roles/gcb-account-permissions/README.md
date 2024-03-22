# 帳號與存取控制

## 手動處理項目

### TWGCB-01-003-0063

* 執行下列指令，確認已存在wheel群組

```bash
grep ^wheel /etc/group | wc -l
```

* 若不存在wheel群組，執行下列指令以建立wheel群組

```bash
groupadd wheel
```

### TWGCB-01-003-0065: 非root系統帳號登入方式

* 執行下列指令，列出所有使用者、UID及shell：

```bash
awk -F： '{print $1 "：" $3 "：" $7}' /etc/passwd
```

* 針對UID小於500之非root系統帳號，執行下列指令進行帳號鎖定與shell設定

```bash
# 將密碼凍結，使該帳號無法登入
usermod -L [username]

# 設定shell
usermod -s /sbin/nologin [username]
```

### TWGCB-01-003-0066: 使用空白密碼之帳號登入方式

* 執行下列指令列出使用空白密碼之帳號

```bash
awk -F: '($2 == "") {print}' /etc/shadow
```

* 若有使用空白密碼之帳號，則執行下列指令以設定密碼：

```bash
passwd [username]
```

### TWGCB-01-003-0067: 所有帳號的密碼遮蔽

* 執行下列指令，確認所有帳號的密碼欄位顯示為x，沒有儲存密碼雜湊於/etc/passwd中：

```bash
awk -F： '($2 != "x") {print}' /etc/passwd
```
* 若有帳號的密碼欄位出現非x值，請執行下列指令進行密碼設定：

```bash
passwd [username]
```

### TWGCB-01-003-0068: UID=0之帳號

* 執行下列指令，列出UID=0之帳號：

```bash
awk -F: '($3 == "0") {print}' /etc/passwd
```

* 若存在非root帳號，可執行下列指令移除帳號或重新設定UID：

```bash
userdel [username]
# 或
usermod -u [UID] [username]
```

### TWGCB-01-003-0073: /etc/group檔案行首的「+」符號

* 執行下列指令，確認/etc/group 檔案行首是否存在「+」符號：

```bash
grep "^+:" /etc/group
```

* 如果有，請編輯/etc/group檔案，將行首為「+」符號之列移除

### TWGCB-01-003-0074: /etc/passwd 檔案行首的「+」符號

* 執行下列指令，確認/etc/passwd 檔案行首是否存在「+」符號：

```bash
grep "^+:" /etc/passwd
```

* 如果有，請編輯/etc/passwd 檔案，將行首為「+」符號之列移除

### TWGCB-01-003-0075: /etc/shadow 檔案行首的「+」符號

* 執行下列指令，確認/etc/shadow 檔案行首是否存在「+」符號：

```bash
grep "^+:" /etc/shadow
```

* 如果有，請編輯/etc/shadow 檔案，將行首為「+」符號之列移除

### TWGCB-01-003-0085: root帳號的路徑變數

* 執行下列指令以顯示 PATH變數內容：

```bash
echo $PATH
```

* 如出現「.」、「..」、路徑開頭不是「/」及空元素等內容，請編輯/etc/profile檔案進行修改

### TWGCB-01-003-0086: root帳號的路徑變數不包含world-writable或group-writable目錄

* 執行下列指令以顯示 PATH變數內容：

```bash
echo $PATH
```

* 如出現具有world-writable權限或group-writable權限之目錄，請編輯/etc/profile檔案進行修改，或執行下列指令變更目錄權限：

```bash
chmod g-w (目錄)
#或
chmod o-w (目錄)
```

### TWGCB-01-003-0087: 使用者家目錄權限

* 針對每個使用者的家目錄執行下列指令：

```bash
ls -ld /home/(使用者帳號)
```

* 如出現具有world-readable或group-writable權限之家目錄，請執行下列指令變更目錄權限：

```bash
chmod g-w /home/(使用者帳號)
chmod o-rwx /home/(使用者帳號)
```

### TWGCB-01-003-0095: 以密碼保護GRUB開機載入程式

* 請先選擇一組密碼，執行下列指令產生雜湊密碼：

```bash
#grub-md5-crypt
Password：(輸入密碼)
Retype Password：(再次輸入密碼)
```

* 編輯/etc/grub.conf檔案，將上一個步驟產生的雜湊密碼，新增或修改成以下內容：

```bash
password --md5 (雜湊密碼)
```

### TWGCB-01-003-0096: 以密碼保護單一使用者模式

* 編輯/etc/inittab檔案，新增或修改成以下內容：

```
~:S:wait:/sbin/sulogin
```