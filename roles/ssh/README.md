# SSH設定

## 手動處理項目

### TWGCB-01-008-0266: 限制存取SSH

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