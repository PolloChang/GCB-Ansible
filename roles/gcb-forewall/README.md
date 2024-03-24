# SELinux

## 手動處理項目

### TWGCB-01-003-0136: 無線網路介面卡

1. 首先執行下列指令，取得網路介面卡資訊，從中找出無線網路介面卡(interface可能為wlan0、eth0或wifi0)：

```bash
ifconfig –a
```

2. 執行下列指令，立即關閉無線網路介面卡：

```bash
ifdown interface
```
3. 上述設定可立即關閉無線網路介面卡，但重新開機後，仍可使用無線網路介面卡。若要以後開機時皆關閉無線網路介面卡，請執行下列指令移除相關設定檔：

```bash
rm /etc/sysconfig/network-scripts/ifcfg-interface
```