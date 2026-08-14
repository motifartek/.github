
<div align="center">
<a href="https://github.com/motifartek"> <img src="https://raw.githubusercontent.com/motifartek/.github/refs/heads/main/assets/logo.png" alt="MotifAI Logo" width="360" /> </a>
<br>
<br>
  <p align="center">
    <strong>Yerel (Offline), Ekstrem Hızlı ve Otonom Multimodal Video Analiz & Karar Destek Sistemi</strong>
  </p>

  <p align="center">
    <a href="#-proje-hakkında">Hakkında</a> •
    <a href="#-temel-mimarı--teknoloji-yığını">Mimari</a> •
    <a href="#-şartname-ve-kpi-uyumluluğu">Şartname Uyumı</a> •
    <a href="#-dizin-yapısı">Dizin Yapısı</a> •
    <a href="#-kurulum-ve-başlangıç">Kurulum</a>
  </p>

  [![TEKNOFEST 2026](https://img.shields.io/badge/TEKNOFEST-2026_Yapay_Zeka_Ajanları-orange?style=for-the-badge)](https://www.teknofest.org)
  [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?style=for-the-badge)](LICENSE)
  [![Rust](https://img.shields.io/badge/Backend-Rust-red?style=for-the-badge&logo=rust)](https://www.rust-lang.org/)

</div>

---

## 📌 Proje Hakkında

**MotifAI**, savunma sanayi tesisleri, saha operasyonları ve iş sağlığı güvenliği (İSG) süreçleri için geliştirilmiş; yüksek hacimli video akışlarını canlı ve geçmişe dönük olarak analiz edebilen **tamamen yerel (offline)** bir yapay zeka ve karar destek sistemidir.

Geliştirilen bu sistem, geleneksel sabit kurallı video analiz teknolojilerinin ötesine geçerek:
- **Sahne Bütünlüğü & Zamansal Farkındalık:** Videoyu sadece tekil kareler olarak değil, zamansal olay akışı (event flow) mantığıyla yorumlar.
- **Otonom Karar Destek:** Tespit edilen riskli durumları ve olayları anında yapılandırılmış (JSON/Graph) formata dönüştürür ve operatör için uygulanabilir eylem önerileri sunar.
- **Sıfır Dış Bağımlılık:** Tüm çıkarım (inference), veri depolama ve iş mantığı süreçlerini hiçbir harici API veya bulut servisine ihtiyaç duymadan kapalı ağda yürütür.

---

## 🏗️ Temel Mimari & Teknoloji Yığını

MotifAI, **Domain-Driven Design (DDD)** prensiplerine sadık kalınarak, yüksek eşzamanlılık (concurrency) ve ultra düşük gecikme (sub-millisecond latency) hedefiyle kurgulanmıştır.
```
                     +-----------------------------------+
                     |   Next.js Dashboard & Landing     |
                     |   (Frontend / Live Monitoring)    |
                     +-----------------+-----------------+
                                       |
                                       | (HTTP / SSE - Realtime Events)
                                       v
                     +-----------------------------------+
                     |      Rust API Gateway (Axum)      |
                     |  (Auth Verification & Router)     |
                     +--------+----------------+---------+
                              |                |
              (Publish/Sub)   |                |  (Auth Check / Local DB)
              +---------------+                +---------------+
              |                                                |
              v                                                v
     +-----------------+                              +-----------------+
     | NATS JetStream  |                              | SQLite & Ory    |
     |   (Event Bus)   |                              |  (Kratos/Keto)  |
     +--------+--------+                              +-----------------+
              |
              | (Async Olay Dağıtımı / Task Queue)
              +-----------------------+-----------------------+
              |                       |                       |
              v                       v                       v
    +------------------+    +------------------+    +-------------------+
    |  apps/stream     |    | ai/orchestrator  |    | ai/inference      |
    |  (Video Ingest & |    | (Agentic Logic & |    | (vLLM Qwen-2VL    |
    | Dynamic Frame)   |    | Tool Call)       |    | Local Model)      |
    +--------+---------+    +--------+---------+    +-------------------+
             |                       |
             | (Raw Video/Chunks)    | (JSON Özetleri, Graf & Vektörler)
             v                       v
    +------------------+    +-------------------------------------------+
    |   MinIO (S3)     |    |              Veri Katmanı                 |
    |  (Blob Storage)  |    |  +-------------------------------------+  |
    +------------------+    |  | SurrealDB (Document & Graph DB)     |  |
                            |  +-------------------------------------+  |
                            |  | Qdrant (Vector DB / RAG Memory)     |  |
                            |  +-------------------------------------+  |
                            +-------------------------------------------+
```

### 🛠️ Teknoloji Seti

- **Core & Backend:** Rust (Axum, Tokio) — *Ekstrem hız ve bellek güvenliği.*
- **AI & Inference Engine:** vLLM üzerinde **Qwen-2VL** — *Multimodal görsel/metinsel akıl yürütme.*
- **Message Broker:** NATS JetStream — *Hafif, yüksek performanslı ve olay güdümlü (event-driven) iletişim.*
- **Databases:**
  - **SurrealDB:** Multi-model (Document + Graph) yerel veritabanı — *Olaylar, nesneler ve risk ilişkilerini saklama.*
  - **Qdrant:** Rust tabanlı Vektör Veritabanı — *Geçmiş olaylar ve özetler üzerinde Semantic RAG.*
  - **SQLite:** Ory Kratos/Keto için ultra hafif kimlik doğrulama katmanı depolaması.
- **Media & Ingestion Processing:** `packages/optics` (FFmpeg wrapper, OpenCV akıllı kare örnekleme).
- **Storage:** MinIO — *Yerel S3 uyumlu nesne depolama.*
- **Frontend:** Next.js, Tailwind CSS, Shadcn UI — *SSE (Server-Sent Events) destekli canlı izleme arayüzü.*

---

## 📋 Şartname ve KPI Uyumluluğu

MotifAI, TEKNOFEST Yapay Zeka Dil Ajanları Yarışması (3. Senaryo) şartnamesinde belirtilen tüm teknik ve etik kriterleri karşılamak üzere tasarlanmıştır:

| Şartname Gereksinimi | MotifAI Çözümü | Durum |
| :--- | :--- | :---: |
| **Tam Yerel (Offline) Çalışma** | Sıfır dış API; vLLM, SurrealDB ve MinIO lokalde orkestre edilir. | ✅ |
| **Yapılandırılmış JSON Çıktısı** | Qwen-2VL + SurrealDB Graph sorguları ile anında şematize edilmiş JSON üretimi. | ✅ |
| **Kritik An ve Zamansal Analiz** | Akıllı kare örnekleme (Dynamic Sampling) ile zaman damgalı olay tespiti. | ✅ |
| **Aksiyon & Risk Önerisi** | Olay durumuna göre otomatik risk seviyesi belirleme ve operatör eylem rehberliği. | ✅ |
| **Açık Kaynak Lisanslama** | Apache License 2.0 ile tam şeffaf ve sürdürülebilir kod altyapısı. | ✅ |

---

## 📁 Dizin Yapısı

```bash
motif-ai/
├── apps/
│   ├── gateway/         # Rust Axum API Gateway & SSE Sunucusu
│   ├── stream/          # Video alma, chunking ve frame işleme servisi
│   ├── ai/
│   │   ├── orchestrator/# Agentic akıl yürütme ve tool çağırım merkezi
│   │   ├── inference/   # vLLM (Qwen-2VL) wrapper ve modeller
│   │   ├── memory/      # Qdrant entegrasyonlu uzun süreli hafıza
│   │   └── knowledge/   # RAG ve bilgi tabanı yönetim servisi
│   ├── dashboard/       # Next.js canlı izleme paneli (Frontend)
│   └── landing/         # Tanıtım ve dokümantasyon sitesi
├── packages/
│   ├── optics/          # Görsel işleme ve medya araç takımı (Rust)
│   └── database/        # SurrealDB & Qdrant ortak istemcileri ve şemaları
├── platform/
│   ├── database/        # SurrealDB & Qdrant konfigürasyonları
│   ├── storage/         # MinIO altyapı kurulumları
│   └── docker-compose.yml
├── tools/               # Benchmark, mock fonksiyon ve test betikleri
└── docs/                # Proje dokümantasyonu ve jüri sunum materyalleri
```
