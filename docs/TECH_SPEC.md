# **B2B CRM — Technical Specification (Final Clean Version)**

# **1) System Architecture Overview**

Bu sistem:

- Web tabanlı (Single Tenant)
- Supabase (PostgreSQL + Auth + RLS)
- Server-side validation zorunlu
- Log sistemi append-only
- Soft delete destekli
- Currency snapshot immutable
- State-machine tabanlı satış süreci
- Snapshot tabanlı finansal bütünlük

# **Temel Prensipler**

- Hiçbir finansal veri sonradan değiştirilemez.
- Approved satışlar kilitlidir.
- Tüm kritik işlemler loglanır.
- Snapshot alanlar immutable’dır.
- Geçmiş veriler kur değişiminden etkilenmez.

# **2) Sale Status State Machine**

# **Sale Status Enum**

draft

pending_approval

approved

rejected

auto_rejected

cancelled

# **State Rules**

- draft → approved
- draft → pending_approval
- pending_approval → approved
- pending_approval → rejected
- pending_approval → auto_rejected
- approved → cancelled (admin only)

Kurallar:

- Approved satış tekrar pending’e dönemez.
- Approved satış fiyatı değiştirilemez.
- State geçişlerinin tamamı loglanır.

# **3) Min Price Approval Engine**

# **Amaç**

Minimum fiyat altı satışları kontrol altına almak ve marj disiplinini korumak.

# **Price Calculation Logic**

Ürün fiyatları USD tutulur.

Satış sırasında:

min_price_try = min_price_usd_snapshot * exchange_rate_snapshot

Kontrol:

if entered_try >= min_price_try:

status = approved

else:

status = pending_approval

# **Pending Approval Rules**

- Stok düşmez
- Ciro hesaplarına girmez
- Dashboard’da ayrı gösterilir
- Admin notification oluşturulur
- Log kaydı oluşturulur (price_below_min)

# **Approval Action**

Admin approve ederse:

- status = approved
- approved_at set edilir
- approved_by set edilir
- stok düşer
- log oluşturulur (approved_by_admin)

Admin reject ederse:

- status = rejected
- log oluşturulur (rejected_by_admin)

# **Timeout Rule**

- 24 saat içinde işlem yapılmazsa:
    - status = auto_rejected
    - auto_rejected_at set edilir
    - log oluşturulur (auto_rejected_timeout)

Bu işlem scheduled function ile çalışır.

# **Admin Override**

Admin minimum fiyat altına doğrudan satış yapabilir.

Kurallar:

- status = approved
- override_reason zorunlu
- override_flag = true
- log_type = admin_override
- raporlarda override ayrı gösterilir

# **4) Currency Conversion Engine**

# **Global Exchange Rate**

Sistem tek aktif USD/TRY kur kullanır.

exchange_rates tablosu:

- id
- rate
- is_active
- created_at
- changed_by

# **Snapshot Mantığı (CRITICAL)**

Satış oluşturulurken aşağıdaki alanlar kaydedilir:

- exchange_rate_snapshot
- min_price_usd_snapshot
- min_price_try_snapshot
- product_name_snapshot
- category_snapshot

Bu alanlar immutable’dır.

Update edilemez.

# **Currency Validation**

Minimum fiyat kontrolü snapshot üzerinden yapılır.

Kur sonradan değişse bile eski satışlar etkilenmez.

# **Exchange Rate Change Rule**

Kur değiştiğinde:

- eski rate inactive olur
- yeni rate active olur
- log oluşturulur (exchange_rate_changed)

Geçmiş satışlar etkilenmez.

# **5) Product Cost & Basic Inventory (MVP)**

# **Amaç**

- Gerçek brüt marj hesaplamak
- Ürün bazlı kârlılık analizi yapmak
- Satıcı performansını doğru ölçmek
- Sistemi gereksiz karmaşıklaştırmamak

# **Product Cost Fields (Products Table)**

- cost_usd (decimal, required)
- cost_last_updated_at
- cost_last_updated_by

Bu alan varsayılan maliyettir.

# **Stock Field (Products Table)**

- stock_quantity (integer)

**Kurallar**

- Stok negatif olamaz.
- Approved sale olunca düşer.
- Cancelled sale olunca geri eklenir.
- Rejected / auto_rejected satış stok etkilemez.

Stok yetersizse satış approved olamaz.

# **Sale Cost Snapshot (CRITICAL)**

Sales tablosuna eklenir:

- cost_usd_snapshot
- exchange_rate_snapshot
- cost_try_snapshot
- gross_profit_try
- gross_profit_percentage

Hesaplama:

cost_try_snapshot = cost_usd_snapshot * exchange_rate_snapshot

gross_profit_try = sale_total_try - cost_try_snapshot

gross_profit_percentage = gross_profit_try / sale_total_try

Kurallar:

- Bu alanlar immutable’dır.
- Approved satışlarda değiştirilemez.

# **Cost Update Rule**

Admin cost_usd güncelleyebilir.

Ancak:

- Eski satışların snapshot maliyeti değişmez.
- Yeni satışlar yeni maliyetle hesaplanır.
- Cost değişimi loglanır (product_cost_updated).

# **6) Log System Design**

# **Log Table Rules**

- Append-only
- Update yok
- Delete yok
- Soft delete uygulanmaz

# **Log Types**

sale_created

sale_updated

status_changed

price_below_min

approved_by_admin

rejected_by_admin

auto_rejected_timeout

admin_override

customer_transfer

exchange_rate_changed

stock_updated

product_cost_updated

# **Log Fields**

- id
- entity_type
- entity_id
- action_type
- old_value (jsonb)
- new_value (jsonb)
- performed_by
- performed_at

# **7) Customer Ownership Isolation (RLS Concept)**

# **Owner Rule**

customers.owner_id zorunludur.

Sales sorgusu:

SELECT * FROM customers

WHERE owner_id = auth.uid()

Sales başka müşteriyi göremez.

# **Transfer Rule**

Sadece admin:

- owner_id değiştirebilir
- transfer loglanır (customer_transfer)
- geçmiş satışlar korunur

# **8) Inquiry & Follow-up Engine**

# **Inquiry Status Enum**

open

contacted

quoted

won

lost

stale

# **Kurallar**

- open inquiry için task zorunlu
- due_date zorunlu
- 30 gün işlem yoksa stale flag
- Inquiry kapanmadan performans tamam sayılmaz

# **9) Cross-Sell Engine (MVP)**

# **Recommendation Order**

1. product_relations (priority asc)
2. same category products

# **Same Category Rule**

- Aynı category_id
- Satılan ürün hariç
- active_for_recommendation = true

# **Recommendation Behavior**

- Satış onay ekranında gösterilir
- Müşteri detay sayfasında gösterilir
- Zorunlu değildir
- Otomatik task oluşturulmaz (MVP)

# **10) Performance Calculation Engine**

# **Revenue**

SUM(sale_total_try)

WHERE status = approved

# **Gross Profit**

sale_total_try - cost_try_snapshot

# **Invoice Count Rule**

- 1 approved sale = 1 invoice
- Aynı gün aynı müşteri olsa bile ayrı satış ayrı invoice sayılır

# **Hybrid Score (Display Only)**

- Revenue weight = 40%
- Gross profit weight = 30%
- Invoice count weight = 30%

Prim hesaplanmaz.

# **11) Soft Delete Policy**

Soft delete uygulanabilir tablolar:

- customers
- products
- categories

deleted_at alanı set edilir.

Satışlar fiziksel olarak silinmez.

# **12) Performance & Scalability Targets**

- 10 concurrent user
- Sale creation < 2 saniye
- Dashboard load < 3 saniye
- Approval timeout job max 1 dakika gecikme
- 10 yıl veri saklama mimarisi

# **13) Data Integrity Rules (Non-Negotiable)**

- Snapshot alanlar update edilemez
- Log silinemez
- Approved sale fiyatı değiştirilemez
- Approved sale maliyeti değiştirilemez
- Pending sale düzenlenebilir
- Override satışlar ayrı flag taşır
- Stok negatif olamaz
