Yaptıklarım

1- Terminalden ping atmaya çalışarak sorunu doğruladım
2- etc dizinindeki dosyaları kontrol ettim ve birşey bulamadım.
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

Yazım hatası yapıp dsn yapıldığında, sistem arka planda libnss_dsn.so adında var olmayan bir kütüphaneyi aramaya kalktı. Onu bulamayınca da süreci başarısızlıkla sonuçlandı.

----------------------------

Linux'ta bir uygulama (örneğin ping veya curl) bir alan adını çözmek istediğinde doğrudan /etc/resolv.conf dosyasına gitmez.

----------------------------

1. O Dosyada Ne Değişti de Sistem Bozuldu?

İçeride çalıştırdığın o gizli script tam olarak şu tek satırlık işlemi yaptı:

/etc/nsswitch.conf dosyasının içinde varsayılan olarak şu satır bulunur:


hosts:          files dns

Bu satır sisteme der ki: "Bir alan adı (örn: google.com) çözümleneceği zaman önce /etc/hosts dosyasına bak (files), orada bulamazsan sonra DNS sunucularına sor (dns)."

Script'in yaptığı tek şey, dosyada geçen dns kelimesini bir harf hatasıyla (typo) dsn olarak değiştirmek oldu (sed -i 's/dns/dsn/g' /etc/nsswitch.conf).

Sistem google.com'u çözmeye çalıştığında:

    /etc/hosts dosyasına baktı (bulamadı).

    Sırada dsn modülüne sormak vardı ama Linux'ta dsn diye bir modül olmadığı için süreç orada tıkandı.

    /etc/resolv.conf dosyasında 8.8.8.8 yazıyor olsa bile, sistem dns aşamasına hiç geçemediği için DNS sunucusuna paket bile atmadı!

2. Günlük Hayatta Karşıma Çıkarsa Nasıl Teşhis Ederim?

Gerçek bir sunucuda veya iş yerinde böyle bir durumla karşılaştığında hayat kurtaran 2 adımlı harika bir test yöntemi vardır:
Adım A: Doğrudan DNS Sorgusu Yap (dig veya nslookup)

Sistemde sorun varken terminalde şu komutu çalıştırırsın:
Bash

dig google.com

dig ve nslookup gibi araçlar nsswitch.conf dosyasını baypas eder, doğrudan /etc/resolv.conf içindeki DNS sunucusuna gider.

    Eğer dig google.com sana IP adresini döndürüyorsa: DNS sunucun ve internetin kesinlikle sağlamdır.

Adım B: Sistem Mantığıyla Sorgu Yap (getent)

Ardından şu komutu çalıştırırsın:
Bash

getent hosts google.com

getent komutu, tıpkı ping veya curl gibi tamamen nsswitch.conf kurallarını takip eder.

    Eğer dig yanıt veriyor ama getent hosts google.com boş dönüyor/hata veriyorsa: Sorun %100 /etc/nsswitch.conf dosyası veya içerisindeki bir kütüphanededir.

    Özetle Teşhis Formülü:
    dig çalışıyor + ping domain çalışmıyor = /etc/nsswitch.conf bozuk!





