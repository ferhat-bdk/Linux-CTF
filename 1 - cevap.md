# Çözümüm


```bash
ping google.com
```

bu kodu girdiğimde bana "temporary failure in name resolution" hatası veriyor. Aklıma DNS çözümleme dosyası geliyor.

etc/hosts dosyasında kayma vardı onu düzelttim hata düzelmedi.

Bağlantıyı kesip tekrar açtım ve düzeldi.

```shell
ifconfig ens33 down # Bağlantıyı kestim
ifocnfig ens33 up # Tekrar aktifleştirdim
```


-----
# Script ne yaptı?

- **Bozucu Script Ne Yaptı?** Bu script, Linux'ta DNS sunucu adreslerinin tutulduğu `/etc/resolv.conf` dosyasının yedeğini aldı ve içine geçersiz bir DNS adresi yazdı (`nameserver 192.0.2.55`). Dolayısıyla sistem `google.com` adresinin IP karşılığını bulamadı. `/etc/hosts` dosyasında yerel IP eşleşmeleri tutulduğu için oradaki düzeltme bu yüzden genel internet erişimini kurtarmadı.
    
- **Senin Çözümün Neden İşe Yaradı?** `ifconfig ens33 down` ve `up` yaparak ağ kartını (arayüzünü) kapatıp açtın. Bu işlem tetiklendiğinde:
    
    - Ağ kartın, bağlı olduğun modeme/yönlendiriciye (DHCP sunucusuna) tekrar bağlandı.
        
    - DHCP sunucun, bilgisayarına otomatik olarak yeni ve **doğru** çalışan DNS bilgilerini tekrar gönderdi.
        
    - Linux da bu yeni gelen doğru DNS bilgilerini otomatik olarak gidip `/etc/resolv.conf` dosyasına yazdı ve eski hatalı ayarın üzerine yazmış oldu!



# Ne öğrendim?

/etc/hosts dosyası lokal bağlantıları /etc/resolv.conf ise dış DNS kayıtlarını tutuyor. Yapay zeka DNS kayıtlarını karıştırınca bu yüzden yolunu bulamadı, bende bağlantıyı sıfırladığım için otomatik olarak DHCP sunucusundan doğru kayıtları çekti
