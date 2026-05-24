# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:** _(Jelaskan dengan kata-katamu sendiri apa fungsi dari CHUNK_SIZE)_

> Jawab: CHUNK_SIZE berfungsi untuk menentukan ukuran potongan teks (chunk) dari dokumen yang akan diproses oleh sistem RAG. Semakin kecil chunk, semakin detail jawaban yang diberikan, sedangkan semakin besar chunk, semakin banyak konteks dalam satu bagian.

**Pengaruhnya:**

| Range Nilai                         | Efek                                                                                                                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kecil (100–200)**                 | _Chunk menjadi lebih spesifik dan lebih akurat, tapi sering kehilangan konteks karena informasi terpecah terlalu kecil._                                |
| **Sedang (250–400)** ⭐ Recommended | _Range ini paling di rekomendasikan karena seimbang antara konteks dan detail dari jawaban. Jadi hasil jawaban lebih relevan dan jawaban lebih lengkap_ |
| **Besar (500–1000)**                | _Terlalu banyak konteks, tetapi jawabannya jadi kurang akurat karena terlalu banyak informasi_                                                          |

**Pertanyaan Refleksi:**

- Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?

  > Jawab: spesifiknya, chunk dengan range sekitar 300 yang lebih tepat, karena konteks dan akurasi itu lebih seimbang.

- Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: jawaban menjadi tidak akurat, parameter yang harus di ubah adalah CHUNK_SIZE. CHUNK_SIZE harus di tingkatkan agar konteks lebih lengkap

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:** _(Jelaskan dengan kata-katamu sendiri apa fungsi dari TOP_K_INITIAL)_

> Jawab: Menentukan chunks yang dipilih pada tahap pencarian awal untuk memastikan informasi yang luas sebelum melalui proses penyaringan sekunder

**Pengaruhnya:**

| Range Nilai                       | Efek                                                                                  |
| --------------------------------- | ------------------------------------------------------------------------------------- |
| **Kecil (5–10)**                  | _Proses lebih cepat, tetapi kemungkinan kehilangan informasi penting_                 |
| **Sedang (15–25)** ⭐ Recommended | _Memberikan keseimbangan antara kecepatan dan kelengkapan kandidat._                  |
| **Besar (30–50)**                 | _Recall lebih tinggi karena banyak kandidat diambil, tetapi waktu proses lebih lama._ |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

- Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.

  > Jawab: TOP-K kecil cukup untuk dataset saya, karena dataset ini hanya membahas 1 topik saja, yaitu RAG. tapi kalau terlalu kecil, bagian penting berpotensi tidak terambil. jadi lebih aman untuk ambil nilai sedang

- Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL menentukan jumlah kandidat awal, sedangkan USE_RERANKER bertugas memilih kandidat yang paling relevan dari hasil tersebut. Jika TOP_K_INITIAL besar tetapi tidak ada reranker, maka banyak chunk yang kurang relevan tetap bisa terpilih. Dengan reranker, hasil dari TOP_K_INITIAL akan disaring ulang agar lebih akurat.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:** _(Jelaskan dengan kata-katamu sendiri apa fungsi dari FINAL_TOP_M)_

> Jawab: FINAL_TOP_M berfungsi untuk menentukan jumlah chunk terbaik yang akan dikirim ke LLM sebagai konteks akhir setelah proses retrieval dan reranking.Parameter ini memastikan hanya informasi yang paling relevan yang diberikan ke model.

**Pengaruhnya:**

| Range Nilai                     | Efek                                                                                                                               |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Sedikit (2–3)**               | _Jawaban menjadi lebih fokus, tetapi ada risiko kehilangan informasi penting dari dokumen seperti penjelasan tambahan tentang RAG_ |
| **Sedang (4–6)** ⭐ Recommended | _Kelengkapan informasi dan fokus jawaban lebih seimbang_                                                                           |
| **Banyak (7–10)**               | _Konteks lebih lengkap, tetapi bisa membuat model bingung karena banyak informasi yang mirip dan meningkatkan waktu proses_        |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

- Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.

  > Jawab: Pada dataset ini, terlalu banyak konteks justru membuat jawaban kurang fokus karena semua chunk membahas RAG dengan isi yang mirip. Jadi lebih baik menggunakan jumlah sedang agar tetap relevan dan tidak membingungkan model.

- Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: Model bisa menghasilkan jawaban yang tidak konsisten atau mencampur informasi dari beberapa bagian, sehingga jawaban menjadi kurang akurat.

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:** _(Jelaskan dengan kata-katamu sendiri apa fungsi dari USE_RERANKER)_

> Jawab: USE_RERANKER berfungsi untuk meningkatkan akurasi pemilihan chunk dengan mengevaluasi ulang relevansi antara pertanyaan dan setiap chunk menggunakan model yang lebih canggih.

**Pengaruhnya:**

| Mode    | Efek                                                                                                      |
| ------- | --------------------------------------------------------------------------------------------------------- |
| `True`  | _Hasil lebih akurat dan relevan karena chunk dipilih dengan lebih tepat, tetapi waktu proses lebih lama._ |
| `False` | _Lebih cepat, tetapi hasil bisa kurang relevan karena hanya mengandalkan similarity sederhana._           |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

- Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.

  > Jawab: Reranker sangat membantu pada pertanyaan yang membutuhkan konteks spesifik, seperti “apa keunggulan RAG dibanding model biasa?” karena harus memilih bagian yang benar-benar relevan, bukan hanya yang mirip secara kata.

- Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: Berdasarkan eksperimen, reranker menambah waktu sekitar 1–2 detik, tergantung jumlah chunk yang diproses, tetapi meningkatkan kualitas jawaban secara signifikan.

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:** _(Jelaskan dengan kata-katamu sendiri apa fungsi dari TEMPERATURE)_

> Jawab: Temperature berfungsi untuk mengatur tingkat kreativitas model dalam menghasilkan jawaban. Semakin rendah nilainya, semakin fokus dan konsisten jawaban terhadap konteks dokumen.

**Pengaruhnya:**

| Range Nilai                          | Efek                                                            |
| ------------------------------------ | --------------------------------------------------------------- |
| **0.0–0.2** ⭐ Recommended untuk RAG | _Jawaban sangat konsisten, faktual, dan sesuai dengan dokumen._ |
| **0.3–0.5**                          | _Jawaban masih cukup akurat, tetapi mulai ada variasi_          |
| **0.6–0.8**                          | _Jawaban lebih kreatif, tetapi mulai keluar dari konteks_       |
| **0.9–1.0**                          | _Jawaban cenderung acak dan berisiko halusinasi_                |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap _grounded_ pada konteks yang diberikan.

**Pertanyaan Refleksi:**

- Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.

  > Jawab: Temperatur ideal adalah sekitar 0.1–0.3 karena jawaban harus akurat dan berdasarkan dokumen, bukan kreatif.

- Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: Jawaban menjadi lebih tidak konsisten dan bisa menambahkan informasi yang tidak ada dalam dokumen.

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model                            | Ukuran         | Pengaruh terhadap Kecepatan & Kualitas                                                                                 |
| -------------------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `llama-3.1-8b-instant` (Default) | 8B parameter   | _Sangat cepat, cocok untuk testing, tetapi jawaban kadang kurang detail_                                               |
| `openai/gpt-oss-120b`            | 120B parameter | _Sangat akurat dan detail, tetapi jauh lebih lambat_                                                                   |
| `llama-3.3-70b-versatile`        | 70B parameter  | _Seimbang antara kecepatan dan kualitas, cocok untuk penggunaan yang membutuhkan akurasi tinggi tanpa terlalu lambat._ |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

- Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.

  > Jawab: Ya, model yang lebih besar memberikan jawaban yang lebih detail dan akurat, tetapi membutuhkan waktu lebih lama. Untuk dataset kecil seperti ini, peningkatan kualitas tidak terlalu signifikan dibanding waktu yang dibutuhkan.

- Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: Model llama-3.3-70b-versatile paling cocok karena memberikan keseimbangan antara kecepatan dan kualitas untuk menjawab pertanyaan tentang konsep RAG yang membutuhkan pemahaman konteks.

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter         | Nilai Default | Fungsi                                                                                                                                                                   |
| ----------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `CHUNK_OVERLAP`   | 60            | _Berfungsi untuk menjaga agar informasi antar chunk tetap terhubung. Jika terlalu kecil, konteks bisa terputus; jika terlalu besar, akan terjadi banyak duplikasi data._ |
| `SCORE_THRESHOLD` | 0.30          | _Menentukan batas minimum kemiripan. Jika terlalu tinggi, banyak informasi penting tidak terambil. Jika terlalu rendah, banyak informasi tidak relevan ikut masuk_       |

---

## 📝 Tips Eksperimen

- Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
- Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
- Catat kapan sistem salah & mengapa
- Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
- Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

- Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
- Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
