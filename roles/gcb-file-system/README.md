# 基礎服務

* version: 1.0

## 手動處理項目

### TWGCB-01-003-0177: 藍芽核心模組

1. 編輯/etc/modprobe.conf檔案，新增或修改成以下內容：

```conf
alias net-pf-31 off
alias bluetooth off
```