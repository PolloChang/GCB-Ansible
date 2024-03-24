# SELinux

## 手動處理項目

### TWGCB-01-003-0117: 標記為unlabeled_t的裝置檔案


1. 執行下列指令，尋找是否存在被標記為unlabeled_t的裝置檔案：

```bash
ls -Z | grep unlabeled_t
```

2. 若有，請執行下列指令針對單一檔案設定標籤：

```bash
chcon -t (標籤名稱) (檔案名稱)
```

3. 如欲針對整個檔案系統重新設定標籤，可執行下列指令：

```bash
touch /.autorelabel
reboot
```