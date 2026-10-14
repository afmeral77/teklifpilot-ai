# TeklifPilot — güvenli teklif taslağı akışı

## Uygulanan akış

1. **Oku:** Form değerleri mevcut `formdanTeklif()` ve oturum doğrulamasıyla okunur. Taslak, kullanıcı oturumu yoksa oluşturulmaz.
2. **Analiz et:** Zorunlu alanlar, KDV onayı, miktar/fiyat ve toplam hesapları kontrol edilir. Tutarlar mevcut hesaplama fonksiyonlarından elde edilir.
3. **Taslak hazırla:** Mevcut Supabase `teklifler` sütunlarına uygun kayıt nesnesi bellekte hazırlanır ve kullanıcıya özet gösterilir. Bu aşamada `insert()` çağrısı yoktur.
4. **Onayla:** Kullanıcı, taslağa özel onay kutusunu işaretleyip “Onayla ve Kaydet” düğmesine basmalıdır.
5. **Uygula:** Uygulama oturumu yeniden doğrular ve kullanıcının aynı kullanıcı olduğunu kontrol ettikten sonra mevcut `teklifler` tablosuna tek bir insert gönderir.
6. **Doğrula:** Başarılı yanıt sonrası belge gösterilir. Ağ sonucu belirsizse yeniden gönderim düğmesi kapatılır; önce kayıt listesi kontrol edilmelidir.

Taslak hazırlanmışken form değişirse eski taslak iptal edilir. İptal ve form düzenleme veritabanına yazma yapmaz.

## Önemli kapsam notu

Depodaki mevcut uygulama statik HTML/JavaScript ve kural/şablon tabanlı açıklama oluşturucudur; bağlı bir LLM veya gerçek AI ajanı bulunmuyor. Bu değişiklik mevcut teklif oluşturma akışına insan onay kapısı ekler. İleride LLM ile analiz eklenirse model yalnızca doğrulanmış girdilerden **taslak/öneri** üretmeli; Supabase yazma yetkisi model aracına verilmemeli, son yazma çağrısı kullanıcı onayı ve sunucu tarafı doğrulamadan sonra yapılmalıdır.

## Yayına almadan önce Supabase güvenlik kontrolü

- `teklifler` tablosunda RLS etkin olmalı.
- `SELECT` ve `INSERT` politikaları yalnızca `authenticated` kullanıcıya izin vermeli ve `auth.uid() = user_id` koşulunu doğrulamalı. UPDATE/DELETE gerekmiyorsa bu izinleri açmayın.
- Mevcut tüm RLS politikalarını gözden geçirin: PostgreSQL'de permissive politikalar OR ile birleşebilir; yeni bir dar politika eklemek başka geniş bir politikayı etkisizleştirmez.
- `user_id` sütunu `auth.users(id)` ile uyumlu olmalı. `anon`/service-role anahtarını tarayıcıya koymayın. Mevcut publishable key istemci tarafında kullanılabilir; güvenlik sınırı RLS'dir.
- Ağ hatası/tekrar gönderim senaryolarına karşı veritabanında uygun bir benzersiz kısıt ve/veya sunucu tarafı idempotency anahtarı değerlendirin. Bu depo tablosunun şeması ve mevcut politikaları verilmediğinden migration otomatik uygulanmadı.

Bu akış bir arayüz onay kapısıdır; güvenlik için Supabase RLS'nin yerine geçmez. Üretim için yazma işlemini sunucu tarafında doğrulayan endpoint'e taşımak önerilir.
