
# Figma to React Project Documentation

Bu doküman, Figma tasarımlarının React bileşenlerine dönüştürülmesini sağlayan GenAI destekli dönüşüm projesi için kapsam, veri ihtiyaçları, pipeline, metrikler, riskler ve proje yapısını içerir.

---

## 1. Kapsam Dokümanı

| Bölüm | Açıklama | Figma→React Örnek İçerik |
|-------|----------|---------------------------|
| 1. Giriş | Projenin genel tanımı ve vizyonu | Figma tasarımlarının kurallı şekilde React (TypeScript) bileşenlerine dönüştürülmesi; tasarım-tabanlı geliştirme süresini %40 azaltmak |
| 2. Mevcut Durum Analizi | Var olan süreçler ve problemler | Elle dönüştürme uzun sürüyor, pixel-parity hataları, farklı ekiplerde tutarsız component kullanımı, tasarım token’larının koda eksik yansıması |
| 3. İş Hedefleri ve KPI'lar | Başarı tanımı ve ölçümü | Kod üretiminde derleme başarı oranı %95+, PR kabul oranı %80+, pixel parity ≥ %90, geliştirici süresinde %40 azalma |
| 4. Kapsam (Dahil) | Dahil olan sistemler ve veriler | Figma API, design tokens (color/typography/spacing), component mapping sözlüğü, React + TypeScript, Storybook, ESLint/Prettier |
| 5. Kapsam Dışı (Şimdilik) | Şimdilik dahil edilmeyecek konular | Backend iş mantığı, karmaşık domain state (Redux/RTK Query) entegrasyonu, mobil (React Native) |
| 6. Stakeholder Haritası | Rol ve sorumluluklar | Design Lead, FE Lead, GenAI Eng, DevOps, Security |
| 7. Zaman Çizelgesi | Fazlar ve sprint planı | POC (2 sprint) → Pilot (3 sprint) → Production (4–6 sprint) |
| 8. Riskler ve Varsayımlar | Olası engeller ve varsayımlar | Figma bileşen isimleri tutarsız; mitigasyon: mapping sözlüğü ve linter kuralları |
| 9. Başarı Metrikleri | Ölçülecek performans göstergeleri | Build pass %, PR kabul oranı, pixel parity, a11y puanı |
| 10. Gelecek Fazlar | Faz 2 ve sonrası | Responsive varyant desteği, i18n, dark mode, otomatik PR açma |

---

## 2. Data İhtiyaçları

| Data İhtiyacı | Detaylar | Figma→React Örnek |
|---------------|----------|--------------------|
| Veri Kaynağı | Sistem/araç ve erişim biçimi | Figma API, design tokens JSON, component library |
| Veri Tipi | Metin/görsel/yapılandırılmış | JSON (node ağacı, styles), görseller (export), metin içerik |
| Veri Hacmi | İlk yükleme ve artış | 100+ component, 20+ sayfa; aylık %10 artış |
| Format ve Yapı | Şema/format | JSON şemaları, mapping JSON, kod şablonları |
| Metadata | Etiketler ve ek bilgiler | Component category, a11y flags, responsive rules |
| Gizlilik & Lisans | KVKK/GDPR/IP | NDA kapsamında tasarım varlıkları |
| Erişim Yetkisi | RBAC ve izinler | Token-based erişim, CI/CD secret management |
| Update Frekansı | İnkremantal yenileme | Günlük component taraması, delta update |
| Kalite Kontrol | Data & çıktı doğrulama | Golden set karşılaştırma, pixel diff, snapshot test |

---

## 3. Pipeline Adımları

| Adım | Detay | Araçlar | Çıktı |
|------|-------|--------|-------|
| Figma Ingestion | Figma API’den veri çekimi | Python/Node, cron jobs | Raw JSON + tokens.json + components.json |
| Ön İşleme & Eşleme | Mapping sözlüğü ve props şeması oluşturma | JSON şema, mapping editor | mapping.json, props_schema.json |
| Şablonlama | Kod şablonlarıyla üretim | Handlebars/EJS, Plop | Component.tsx, styles.css |
| LLM Destekli Dönüşüm | Edge case layout çözümleri | LLM API, prompt library | Düzenlenmiş TSX/CSS |
| Doğrulama & Test | Unit, snapshot, pixel parity testleri | Jest, Playwright, Loki | Test raporları |
| Storybook & Demo | İzole önizleme | Storybook | Stories ve dokümantasyon |
| CI/CD & PR | Otomatik PR ve kalite bariyerleri | GitHub Actions | PR, onay ve merge |

---

## 4. Metrikler

| Metrik | Tanım | Hedef | Toplama Yöntemi |
|--------|-------|-------|-----------------|
| Build Pass % | Üretilen kodun CI’da derlenme oranı | ≥ 95 | CI pipeline sonucundan |
| PR Kabul Oranı % | Açılan PR’ların kabul edilme oranı | ≥ 80 | Repo istatistikleri |
| Pixel Parity Skoru | Tasarım ile render arasındaki görsel benzerlik | ≥ 0.90 | Görüntü fark (SSIM/PSNR) |
| A11y Puanı | Lighthouse/axe erişilebilirlik skoru | ≥ 90 | Lighthouse/axe raporu |
| Geliştirici Süresi | Manuel dönüşüme göre saat tasarrufu | -%40 | Zaman log’ları |
| Test Pass % | Unit ve snapshot test geçme oranı | ≥ 95 | CI test raporları |

---

## 5. Riskler ve Mitigasyon

| Risk | Etki | Mitigasyon | Sahip |
|------|------|------------|-------|
| Figma component isimleri tutarsız | Mapping hataları | İsimlendirme rehberi + otomatik linter | Design Lead |
| Eksik/yanlış design tokens | UI tutarsızlığı | Token audit + source of truth | Design Systems |
| LLM yanlış layout üretir | UI kırılması | Guardrails, şablon-first, pixel diff test | GenAI Eng |
| A11y ihlalleri | Uyumsuzluk ve yasal risk | Axe/Lighthouse bariyerleri | FE Lead |
| Gizlilik/Lisans sorunları | IP ihlali | NDA, asset lisans kontrolü | Legal/Sec |

---

## 6. Zaman Çizelgesi

| Faz | Süre | Aktiviteler | Çıktılar |
|-----|------|-------------|----------|
| POC | 2 sprint | Altın set (5 component), ingestion + mapping | İlk kod üretimi, pixel parity raporu |
| Pilot | 3 sprint | Varyantlar, a11y, Storybook, CI bariyerleri | PR akışı ve kabul edilen bileşenler |
| Production | 4-6 sprint | Delta update, performans optimizasyonu | Stabil pipeline + metrik hedefleri |

---

## 7. Proje Yapısı

```
/F2R_Project
  ├── 01_Business_Case/
  │   ├── Vision_Statement.md
  │   ├── ROI_Analysis.xlsx
  │
  ├── 02_Project_Scope/
  │   ├── Scope_Document.md
  │   ├── Stakeholder_Map.xlsx
  │   └── Risk_Register.xlsx
  │
  ├── 03_Data/
  │   ├── Data_Sources.md
  │   ├── Tokens.schema.json
  │   └── Components.schema.json
  │
  ├── 04_Model_Pipeline/
  │   ├── mapping.json
  │   ├── props_schema.json
  │   └── templates/
  │
  ├── 05_Evaluation/
  │   ├── golden_set/
  │   └── metrics_dashboard.xlsx
  │
  ├── 06_Implementation/
  │   ├── DevOps_Pipeline.yml
  │   ├── eslint.config.js
  │   └── storybook/
  │
  └── 07_Security_Compliance/
      ├── GDPR_Compliance.md
      └── Access_Control.md
```

