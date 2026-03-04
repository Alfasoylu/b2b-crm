# **🟠 Document Information**

Version: v1.0

Last Updated: 03.03.2026

Owner: Alperen

Status: In Development

Scope: Internal Single Company Use (Single Tenant)

Currency Base (Product Prices): USD

Sales Currency: TRY

Primary Goal: Satış performansını ölçen, müşteri kaçmasını engelleyen ve 10 yıllık veri varlığı oluşturan CRM

# **🟠 1) Problem**

CRM olmadığı için:

- Aylık ortalama 10 satış takipsizlik nedeniyle kaybediliyor.
- Aylık en az 10 “yok denilen ürün” unutuluyor ve geri dönüş yapılmıyor.
- Satıcı başına haftalık ~20 follow-up yapılmıyor (proaktif satış yok).
- Son satış fiyatı ve geçmiş sistematik tutulmadığı için marj kaçıyor.
- Tekrar satış oranı düşük çünkü müşteri takibi ve temas planı sistematik değil.
- Müşteri sahipliği net olmadığı için müşteri kaçışı ve müşteri “çalınması” riski var.
- Veri birikimi yok (10 yıl sonra stratejik veri hazinesi hedefi için temel yok).

# **🟠 2) Goals (KPI)**

90 gün hedef KPI’ları (ölçülebilir):

- Tekrar satış oranı: %10 → %20
- “Yok denilen ürün” → satış dönüşüm oranı: %30’u 60 gün içinde satışa dönüşsün
- Ortalama follow-up süresi: 7 gün → 2 gün
- Min fiyat altı satışların loglanması: %100 loglansın (silinemez)

Performans modeli (hibrit):

- Satıcı performansı; Ciro + Marj + Fatura sayısı hibrit model ile ölçülür.
- Yılda en az 10 kez fatura kesilen müşteri sayısı: 3 → 50 (müşteri kalıcılığı metriği)

# **🟠 3) Non-Goals**

Bu proje şunları yapmayacaktır:

- Muhasebe modülü yok
- ERP seviyesinde stok yönetimi yok
- Mobil native uygulama yok (web/PWA yeterli)
- Çoklu firma (multi-tenant) desteği yok
- Prim/komisyon hesaplama motoru yok (manuel hesaplanacak)

# **🟠 4) Users & Roles**

**Admin**

Erişim

- Tüm müşterileri görür
- Tüm satışları görür
- Tüm yok-denilen ürünleri görür
- Tüm görevleri görür
- Tüm logları görür
- Tüm satıcı performansını görür

Yetki

- Satış fiyatı ekleyebilir/değiştirebilir
- Min fiyat altına düşebilir (admin_override, log zorunlu)
- Ürün ekleyebilir/düzenleyebilir/silebilir
- Kategori/etiket/ilişkili ürün tanımlayabilir
- Kullanıcı oluşturabilir ve rol atayabilir
- Müşteri owner transferi yapabilir (log zorunlu)
- Görev oluşturabilir ve atayabilir
- CSV export yapabilir

Kısıt

- Silme işlemleri soft-delete
- Min fiyat altı satışta zorunlu açıklama
- Log kayıtlarını silemez

**Sales**

Erişim

- Sadece kendi atanan (owner) müşterilerini görür
- Kendi satışlarını görür
- Kendi görevlerini görür
- Ürünleri görür
- Son satış fiyatını görür
- Pending approval durumundaki kendi satışlarını görür
- Kendi performansını görür

Yetki

- Satış oluşturabilir
- Yok-denilen ürün (inquiry) girebilir
- Yeni müşteri ekleyebilir (otomatik kendisine atanır)
- Göreve not ekleyebilir
- Görevi tamamlandı işaretleyebilir

Kısıt

- Satış fiyatını düzenleyemez (yalnızca oluşturur)
- Min fiyat altına inemez (onay süreci gerekir)
- Başka sales’in müşterisini göremez
- Logları göremez
- CSV export yapamaz
- Ürün silemez

**Warehouse**

Erişim

- Ürün listesini görür
- Stok durumunu görür
- Satış kayıtlarını fiyat hariç görür
- Müşteri listesini isim/telefon seviyesinde görür

Yetki

- Stok güncelleyebilir
- Ürün stok alarmı koyabilir
- Sevkiyat notu ekleyebilir

Kısıt

- Satış fiyatını göremez / değiştiremez
- Müşteri düzenleyemez
- Görev oluşturamaz
- CSV export yapamaz
- Log göremez

**Log Integrity Rule (Değişmez Kural)**

- Hiçbir rol log silemez
- Admin bile log silemez
- Log tablosu append-only çalışır
- Güncelleme yerine yeni kayıt oluşturulur

# **🟠 5) Core Workflows**

**5.1 Sale Creation & Price Control Workflow**

Amaç

Marj kaybını engellemek ve min fiyat altı satışları onay mekanizmasına bağlamak.

Ürün fiyatları (USD alanları)

- Son kullanıcı fiyatı (USD)
- Toptan fiyat (USD)
- Minimum fiyat (USD)

Normal satış

- Entered TL Price ≥ Min TL (kur ile hesaplanan)
→ Status = APPROVED
→ Satış aktif olur
→ Stok düşer
→ Log oluşturulur

Min fiyat altı

- Entered TL Price < Min TL
→ Status = PENDING_APPROVAL
→ Satış raporlara “ciro” olarak girmez
→ Stok düşmez
→ Admin onayına düşer
→ Log oluşturulur

Admin onayı

- Approve → Status = APPROVED, stok düşer, satış aktif olur, log oluşur
- Reject → Status = REJECTED, log oluşur

Timeout

- 24 saat işlem yapılmazsa → AUTO_REJECTED, log oluşur

Admin override

- Admin min fiyat altına doğrudan satış yapabilir
- Log + zorunlu açıklama
- Override satışlar raporlarda ayrı işaretlenir

Sale status enum

- draft
- pending_approval
- approved
- rejected
- auto_rejected

**5.2 Currency Conversion Workflow**

Amaç

USD bazlı ürün fiyatlarının TL satış sırasında otomatik ve tutarlı kontrol edilmesi.

Kurallar

- Ürün fiyatları USD olarak saklanır
- Satış TL olarak girilir
- Sistem global USD/TRY kuru kullanır
- Minimum fiyat kontrolü TL karşılığı üzerinden yapılır
- Kur satış anında snapshot olarak kaydedilir ve sonradan değiştirilemez
- Kur değişimleri loglanır (old_rate, new_rate, changed_by, timestamp)

**5.3 Inquiry Follow-up Workflow (Customer Retention Engine)**

Amaç

Eski talepleri sistematik şekilde takip edip satışa dönüştürmek, müşteri kaçmasını engellemek.

Kural seti

- “Yok denilen ürün” veya “fiyat/erteledi/rakip” gibi talepler inquiry olarak kaydedilir
- Her inquiry için follow-up görevi oluşturulur (due date zorunlu)
- 7 gün işlem yoksa → uyarı
- 30 gün işlem yoksa → stale flag
- Inquiry kapanmadan (closed/won/lost) rapor tamam sayılmaz

**5.4 Cross-Sell Recommendations (MVP)**

Amaç

Satış anında ilgili ürün önererek tekrar satış ve sepeti büyütmek.

Öneri kaynakları

- Related products (admin manuel tanımlar, öncelikli)
- Aynı alt kategori ürünleri (satılan ürün hariç, öneriye açık ürünler)

Gösterim

- Satış onay ekranında
- Müşteri detay sayfasında

Zorunlu değildir. Otomatik görev yok (Phase 2+).

# **🟠 6) Sales Performance Model (Hybrid)**

Amaç

Satıcıyı sadece ciroya değil, sağlıklı büyümeye göre ölçmek.

Ana metrikler

1. Ciro (Revenue) – Onaylı satışların toplamı (TL)
2. Brüt marj (Gross Profit) – (Sale TL − Min TL karşılığı)
3. Fatura sayısı (Invoice Count) – Onaylı satış adedi

Müşteri kalıcılığı metriği

- Yılda ≥10 fatura kesilen müşteri sayısı (retention/bağlılık göstergesi)

Erişim

- Admin: tüm satıcılar
- Sales: yalnızca kendi metrikleri

Not: Prim hesaplaması yok. Sadece performans görünürlüğü.

# **🟠 7) Customer Ownership & Retention Rules**

Amaç

Müşteri kaçmasını engellemek ve müşteri databse’i büyütmek.

Kurallar

- Her müşteri bir owner_id taşır
- Sales yalnızca kendi owner olduğu müşteriyi görür
- Müşteri transferi yalnızca Admin yapabilir
- Transfer işlemi loglanır
- Owner değişse de geçmiş satışlar korunur

# **🟠 8) Data Governance & Long-Term Strategy (10 Yıl Veri)**

Amaç

Sistemi 10 yıllık veri varlığına dönüştürmek.

Kurallar

- Satışlar fiziksel silinmez, soft-delete uygulanır
- Loglar append-only, silinemez
- Kur snapshot değiştirilemez
- Günlük otomatik backup (operasyonel hedef)
- Veri en az 10 yıl saklanacak şekilde tasarlanır

# **🟠 9) Data Model (High-Level Entities)**

- users
- customers
- products
- categories
- product_relations
- product_tags
- product_tag_relations
- sales
- sale_approvals
- inquiries
- tasks
- activity_logs
- exchange_rates

# **🟠 10) Phases**

Phase 1 – Sales Core + Approval + Currency + Logs + Basic Dashboards

Phase 2 – Inquiry + Follow-up Engine + Retention Dashboards

Phase 3 – Cross-sell automation + Proforma + CSV + Attachments

Phase 4 – Optimization + Advanced Reporting

# **🟠 11) Success Criteria (Operational)**

- 10 kullanıcı eş zamanlı kullanımda sorunsuz çalışır
- Veri kaybı yaşanmaz (backup + soft-delete)
- Min fiyat altı satışlar %100 loglanır
- Sales izolasyonu bozulmaz (müşteri çalma engellenir)
