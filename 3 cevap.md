Linux'ta bir dizin, aslında sadece o dizinin içindeki dosyaların listesini tutan bir "adres defteri" gibidir. `/tmp/ctf4/` dizinini düşünelim:

1. **Dolu Durum:** Dizini `tmpfs` (RAM üzerindeki bir dosya sistemi) ile "mount" etmeden önce, dizin içerisinde `gizli_veriler.txt` dosyası mevcuttu. İşletim sistemi, `ls` komutunu çalıştırdığında bu dizin içindeki dosya listesini tarar ve dosyayı bulur.
    
2. **Mount Operasyonu:** `mount -t tmpfs tmpfs /tmp/ctf4/` komutunu girdiğimizde, sistem bu dizinin üzerine **tamamen yeni ve boş bir dosya sistemi katmanı** örter.
    
3. **Sonuç:** `ls` komutu artık dizine baktığında, alt katmandaki (dosyanın olduğu) gerçek dizine değil, üzerine serdiğimiz o yeni, boş `tmpfs` katmanına bakar. Dosya aslında silinmemiştir, hala diskte/bellektedir ve hala oradadır; ancak **"gizlenmiştir" (obscured)**. Sanki bir kitabın üzerine bir kağıt yapıştırmışsın gibi, alttaki yazıyı göremezsin ama kağıdı kaldırdığında (yani `umount` yaptığında) yazı oradadır.

Bazı kodlar

# ----------------------------------------------------
# 1. DOSYA ARAMA KOMUTLARI
# ----------------------------------------------------
find . -name "*.txt"                # Şu anki klasörde .txt dosyalarını arar
find . -iname "*.txt"               # Büyük/küçük harf duyarsız .txt arar (Önerilen)
find /path/to/dir -name "*.txt"     # Belirli bir klasör içinde .txt arar

# ----------------------------------------------------
# 2. MOUNT EDİLMİŞ ALANLARI GÖRÜNTÜLEME
# ----------------------------------------------------
df -h                               # Mount edilen yerleri ve doluluk oranlarını okunabilir gösterir
lsblk                               # Diskleri ve mount noktalarını ağaç yapısında listeler
mount | column -t                   # Tüm mount detaylarını düzenli bir tablo halinde verir
findmnt                             # Modern ve renkli mount ağacı gösterir

# ----------------------------------------------------
# 3. UNMOUNT VE ZORLAMA (BUSY HATASI ÇÖZÜMLERİ)
# ----------------------------------------------------
cd ~                                # Önce meşgul etmemek için hedef klasörden çıkın!

sudo umount -l /tmp/ctf4            # LAZY UNMOUNT: Arka planda güvenli ve hızlıca söker (İlk tercih)
sudo umount -f /tmp/ctf4            # FORCE UNMOUNT: Ağ paylaşımları ve inatçı diskleri zorlar

sudo fuser -mvk /tmp/ctf4           # KESİN ÇÖZÜM: Klasörü kilitleyen tüm işlemleri zorla öldürür
sudo umount /tmp/ctf4               # İşlemleri öldürdükten sonra temiz bir şekilde unmount eder
