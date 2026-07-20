Yaptıklarım

1- Terminalden ping atmaya çalışarak sorunu doğruladım
2- etc dizinindeki dosyaları kontrol ettim ve birşey şey bulamadım.
3- Bu işlem esnasında uğradığı dosyaları görüntületebiliyor muyum diye araştırırken strach komutunu öğrendim
4- Aşağıdaki Komutu ve altındaki komutları girerek adımları analiz ettim
```
strace -e trace=open,openat ping google.com
strace ping google.com
```
5- ilk 11 satırı incelerken nsswitch.conf diye bir dosya fark ettim ve resolv.conf dan önce uğruyordu. İçeriğinde birşey bulamadım.
6- Yapay zekadan ipucu istediğimde bir konfigürasyon dosyası olduğunu belirtti. Anladım ki ilk gelen nsswitch dosyası
7- İçeriğindeki 
```
hosts: files mdsn4_minimal [NotFOUND=return] dsn
```
yazısındaki dsn ibaresi dns olacakmış. Yani dns ibaresi göremezse bilgisayara, dns ibaresini görüyorsa dns sunucusuna gidiyormuş.

-----------------------

Yapay zeka cevabı:

NSS (Name Service Switch) Mantığı:

Linux'ta C kütüphanesi (glibc), isim çözme işlemini yaparken NSS modüllerini kullanır.

/etc/nsswitch.conf dosyasındaki hosts: satırında:

    files: Sistemi /lib/x86_64-linux-gnu/libnss_files.so modülünü yüklemeye ve /etc/hosts dosyasını okumaya yönlendirir.

    dns: Sistemi /lib/x86_64-linux-gnu/libnss_dns.so modülünü yüklemeye ve /etc/resolv.conf içindeki DNS sunucularını sorgulamaya yönlendirir.

Yazım hatası yapıp dsn yapıldığında, sistem arka planda libnss_dsn.so adında var olmayan bir kütüphaneyi aramaya kalktı. Onu bulamayınca da süreci başarısızlıkla sonlandırıp /etc/resolv.conf dosyasına hiç geçemedi.






