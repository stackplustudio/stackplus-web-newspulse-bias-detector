# NewsPulse – AI-Powered News Media with Bias Detection

## 1. Background / Context
Masyarakat kesulitan membedakan berita faktual, opini, atau propaganda. Portal berita tradisional tidak menyediakan alat untuk melihat bias politik atau clickbait. Literasi digital rendah, hoaks menyebar cepat.

## 2. Problem Statement
- Pembaca tidak tahu apakah suatu artikel cenderung ke kiri/kanan atau netral.
- Judul clickbait meningkatkan engagement tapi merusak kepercayaan.
- Tidak ada metrik objektif untuk mengukur kualitas jurnalistik (misal: jumlah sumber, kutipan asli).
- Iklan programmatic mendorong konten sensasional, bukan informatif.

## 3. Goals / Objectives
- Memberikan skor bias politik (kiri/kanan/netral) dengan akurasi >80% dibanding panel ahli.
- Mendeteksi clickbait headline (precision >85%).
- Menyediakan "fact-check sidebar" otomatis untuk klaim kontroversial menggunakan RAG.
- Meningkatkan retensi pembaca yang mengaktifkan fitur ini sebesar 40%.

## 4. Proposed Solution
Platform berita yang menggunakan NLP (fine-tuned BERT) untuk analisis bias dan clickbait. Ditambah RAG pipeline untuk fact-check dengan sumber terpercaya.

## 5. System Design / Architecture
[Artikel dari RSS/API] → [Preprocessing] → [NLP Pipeline]
├→ Bias Classifier
├→ Clickbait Detector
└→ Claim Extractor
↓
[RAG: retrieve dari database berita & Wikipedia]
↓
[Frontend (Next.js)] → [User membaca + insight]

**Komponen:**
- Model fine-tuned: `indobias` (dataset dari berita Indonesia), `clickbait-indonesia`.
- Vector DB: Pinecone / Qdrant untuk RAG.
- Backend: FastAPI (Python) + Celery untuk async processing.

## 6. Detailed Design
**Bias detection:**
- Gunakan model `roberta-base` fine-tuned dengan 10k artikel yang dilabel ahli (kiri, kanan, netral).
- Fitur tambahan: frekuensi kata khas (misal "reformasi" vs "stabilitas"), entity sentiment terhadap tokoh publik.

**Clickbait detection:**
- Ciri: exaggeration, curiosity gap, imperative mood.
- Model klasifikasi binary (clickbait / not) dengan F1 target 0.85.

**Fact-check RAG:**
- Ekstrak klaim (kalimat yang mengandung fakta kuantitatif atau pernyataan kontroversial) dengan dependency parsing.
- Retrieve dokumen relevan (berita terverifikasi dari Antara, Reuters, Wikipedia) menggunakan semantic search.
- LLM (GPT-4o mini / Llama 3) membandingkan klaim dengan evidence → output "Benar", "Salah", "Tidak cukup bukti".

**Edge cases:**
- Sarkasme atau irony → bias detector bisa salah. Beri peringatan "Hasil ini mungkin tidak akurat karena gaya bahasa figuratif".
- Artikel pendek (<100 kata): default ke "netral" dengan confidence rendah.

## 7. Trade-offs & Alternatives
| Alternatif | Kelebihan | Kekurangan |
|------------|-----------|-------------|
| Gunakan API pihak ketiga (seperti Google Perspective) | Mudah | Tidak spesifik bias politik Indonesia, mahal |
| Crowdsourced bias labeling | Partisipasi publik | Tidak konsisten, bisa dimanipulasi |

**Dipilih fine-tuned model sendiri** agar bisa di-deploy offline dan hemat biaya.

## 8. Risks & Mitigation
| Risiko | Mitigasi |
|--------|-----------|
| Model bias karena data latih tidak seimbang | Oversampling kelas minoritas, evaluasi dengan confusion matrix |
| Klaim tidak terdeteksi karena kalimat kompleks | Fallback: user bisa manual highlight teks untuk di-fact-check |
| Serangan adversarial (artikel sengaja dibuat membingungkan model) | Regular retraining dengan data baru + adversarial training |

## 9. Metrics / Evaluation
- **Bias accuracy**: Cohen's Kappa dengan 3 labeler manusia ≥ 0.75
- **Clickbait detection**: Precision 85%, Recall 80%
- **Fact-check latency**: < 3 detik per klaim (p95)
- **User trust survey**: 70% pembaca merasa lebih percaya setelah pakai fitur

## 10. Implementation Plan
- [ ] Minggu 1-2: Koleksi dataset bias & clickbait (scrape + label manual 2000 artikel)
- [ ] Minggu 3: Fine-tune RoBERTa (gunakan GPU colab / AWS SageMaker)
- [ ] Minggu 4: Bangun RAG pipeline dengan Qdrant & LLM
- [ ] Minggu 5: Integrasi ke Next.js frontend (plugin / website)
- [ ] Minggu 6: Beta test dengan 100 pengguna, iterasi

**Tech stack:** Python (PyTorch, Transformers), FastAPI, Qdrant, Next.js, Redis (cache hasil).
**Kompleksitas:** Hard
**Role:** AI Engineer (2), Backend (1), Frontend (1), Data Labeler (part-time)