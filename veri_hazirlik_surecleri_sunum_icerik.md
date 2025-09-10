
# Veri Hazırlık Süreçleri ve Proje Kapsamı - Detaylı Sunum İçeriği

Bu doküman, veri hazırlık süreçlerini ve kapsam belirleme adımlarını yazılım ekiplerine detaylı olarak anlatmak için hazırlanmıştır.
Her bir slaytta **neler anlatılacağı** ve sunum sırasında **hangi mesajların verilmesi gerektiği** açıklanmıştır.

---

## Slide 1 – Başlık
**Veri Hazırlık Süreçleri ve Proje Kapsamı**  
*"Doğru veri, doğru model, doğru ürün"*

**Anlatım İçeriği:**
- Bu sunumun amacı, veri hazırlık süreçlerini yazılım ekiplerine tanıtmak.
- Veri hazırlığın projelerin neden en kritik aşaması olduğunu ve neden zaman aldığını açıklamak.
- Yazılım ekiplerinin bu süreçlere nasıl katkıda bulunabileceğini göstermek.
- Sunumun sonunda kapsamın nasıl hazırlanacağı, hangi dokümanların oluşturulacağı ve veri hazırlık sürecinin yazılım geliştirme yaşam döngüsü ile nasıl entegre olacağı netleşmiş olacak.

---

## Slide 2 – Neden Veri Hazırlık?

**Anlatım İçeriği:**
- Bir makine öğrenmesi veya veri bilimi projesinde modelin başarısı, doğrudan verinin kalitesi ile ilişkilidir.
- Eğer veri eksik, yanlış veya tutarsızsa, model ne kadar iyi olursa olsun sonuç güvenilir olmayacaktır.
- Çoğu AI/ML projesinde toplam sürenin **%70’i veri hazırlık süreçlerine** gider. Bu oran, veri hazırlığın ne kadar kritik bir iş yükü olduğunu gösterir.
- "Çöp veri → çöp sonuç" prensibi: Yanlış veriye dayalı analiz, yanlış iş kararlarına yol açar.
- Veri hazırlık, teknik bir görevden öte **stratejik bir adımdır**.

---

## Slide 3 – Veri Hazırlık Süreci Genel Akış

**Anlatım İçeriği:**
- Veri hazırlık birbiriyle bağlantılı bir dizi adımdan oluşur.
- Adımlar lineer görünse de aslında **döngüsel bir süreçtir**; bir adımda çıkan bir sorun önceki adıma dönmeyi gerektirebilir.
- Süreç adımları:
  1. Veri Kaynaklarının Belirlenmesi
  2. Veri Toplama ve Erişim
  3. Veri Temizleme ve Düzenleme
  4. Veri Zenginleştirme ve Dönüştürme
  5. Metadata ve Dokümantasyon
  6. Model Pipeline Entegrasyonu

> Amaç: En baştan kaliteli, güvenilir ve izlenebilir veri elde etmek.

---

## Slide 4 – Adım 1: Veri Kaynaklarının Belirlenmesi

**Anlatım İçeriği:**
- Projenin temeli hangi veriyle çalıştığımıza bağlıdır. Bu nedenle **doğru veri kaynağının belirlenmesi kritik** bir adımdır.
- Bu aşamada:
  - Hangi sistemlerden veri alacağımız belirlenir (CRM, ERP, mobil uygulama logları vb.).
  - Verilerin sahipliği netleştirilir.
  - Hassas veriler belirlenir (PII, KVKK, GDPR kapsamındaki bilgiler).
  - Erişim izinleri için gerekli süreçler başlatılır.
- Yanlış veya eksik belirlenen veri kaynağı, projenin baştan yanlış yönlendirilmesine sebep olur.

---

## Slide 5 – Adım 2: Veri Toplama ve Erişim

**Anlatım İçeriği:**
- Belirlenen kaynaklardan verilerin çekilmesi aşamasıdır.
- Yapılacak işler:
  - API, batch job veya manuel export yöntemleriyle verinin alınması.
  - ETL veya ELT süreçlerinin tasarlanması.
  - Büyük veriyle çalışırken sampling yapılması.
- Zorluklar:
  - Veri silo'larda olabilir.
  - Erişim izin süreçleri uzun zaman alabilir.
  - Büyük veri transferi zaman alabilir.

**İpucu:** Erişim sürecine proje başlar başlamaz başlanmalı.

---

## Slide 6 – Adım 3: Veri Temizleme ve Düzenleme

**Anlatım İçeriği:**
- Çoğu ham veri doğrudan kullanılabilir kalitede değildir.
- Bu aşamada:
  - Eksik veriler tespit edilir ve yönetilir.
  - Hatalı veya uç değerler düzeltilir.
  - Duplicate kayıtlar kaldırılır.
  - Veri formatları standardize edilir.
- Örnek:
  - `email` alanındaki boş kayıtlar model doğruluğunu %20 düşürebilir.
- Bu aşama doğru yapılmazsa tüm model yanlış sonuçlar üretir.

---

## Slide 7 – Adım 4: Veri Zenginleştirme ve Dönüştürme

**Anlatım İçeriği:**
- Veri tek başına yeterli değildir, modele uygun hale getirilmelidir.
- Bu aşamada feature engineering yapılır.
  - Ham veriden model için anlamlı özellikler çıkarılır.
  - Farklı veri kaynakları birleştirilir.
- Örnekler:
  - Tarih verisinden hafta içi/haftasonu bilgisi üretmek.
  - Satın alma geçmişinden müşteri segmentleri oluşturmak.
  - Loglardan oturum süresi hesaplamak.

---

## Slide 8 – Adım 5: Metadata ve Dokümantasyon

**Anlatım İçeriği:**
- Veriyle ilgili bilgilerin dokümante edilmesi önemlidir.
- Metadata ekipler arasında ortak bir dil sağlar.
- Her veri alanı için şu bilgiler olmalıdır:
  - Anlamı
  - Tipi
  - Kaynağı
  - Kullanım alanı
- Örnek tablo:

| Alan | Açıklama | Tip | Kaynak |
|------|----------|-----|--------|
| user_id | Kullanıcı ID | Integer | CRM DB |
| signup_date | Kayıt tarihi | Date | Mobile App |

> Dokümantasyon olmazsa, farklı ekipler aynı veriyi farklı yorumlar.

---

## Slide 9 – Veri Hazırlığın Uzun Sürmesinin Sebepleri

**Anlatım İçeriği:**
1. Veri karmaşık ve dağınık olabilir.
2. Erişim izinleri zaman alır (KVKK, GDPR).
3. Veri kalitesi düşük olabilir (eksik, hatalı, tutarsız).
4. Yanlış bir adımın geri dönüş maliyeti yüksektir.
5. Ekipler arası iletişim eksikliği veri yorumlama hatalarına yol açar.

> *Veri hazırlık ne kadar iyi yapılırsa, proje o kadar hızlı ve güvenilir ilerler.*

---

## Slide 10 – Kapsam Nasıl Hazırlanır?

**Anlatım İçeriği:**
- Kapsam, veri hazırlığın sınırlarını belirler.
- Adımlar:
  1. İş hedeflerini netleştir.
  2. Kullanıcı hikayelerini çıkar.
  3. Veri kaynaklarını belirle.
  4. Başarı metriklerini tanımla.
  5. Dahil/Dışında kalanları belirle.
  6. Riskleri ve varsayımları dokümante et.
  7. Pilot faz veri setini seç.

**Örnek:**
- Dahil: CRM verisi, mobil uygulama logları.
- Hariç: Sosyal medya verisi, üçüncü parti API.

---

## Slide 11 – Kapsam Dokümanı Yapısı

**Anlatım İçeriği:**
- Kapsam dokümanı ekipler için ortak referanstır.
- İçermesi gereken bölümler:
  1. Giriş (Projenin amacı, vizyonu)
  2. Mevcut Durum (Var olan sistemler ve sorunlar)
  3. İş Hedefleri ve KPI
  4. Veri Kaynakları
  5. Kapsam Dahil/Dışında
  6. Riskler ve Varsayımlar
  7. Zaman Çizelgesi (POC → Pilot → Production)

---

## Slide 12 – Yazılım Ekipleri İçin Öneriler

**Anlatım İçeriği:**
- Yazılım ekipleri veri hazırlık süreçlerine **erken dahil olmalı**.
- API ve log tasarımı data science ekibiyle birlikte planlanmalı.
- Veri hazırlık aşaması sabır ister; ilk başta yavaş ilerler ama sonrasında hız kazandırır.
- Dokümantasyon her zaman güncel tutulmalı.
- Temiz veri → doğru analiz → güvenilir ürün.

---

## Genel Mesaj

Veri hazırlığı, yazılım ekiplerinin aktif katılımı olmadan başarıya ulaşamaz.  
Bu süreç doğru yönetildiğinde, modelin başarısı ve ürünün güvenilirliği doğrudan artar.

