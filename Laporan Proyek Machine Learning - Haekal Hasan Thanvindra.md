# Laporan Proyek Machine Learning - Haekal Hasan Thanvindra
---
## **Project Overview**

### **Latar Belakang**
Board game merupakan bentuk hiburan yang telah ada selama ribuan tahun dan terus berkembang hingga kini. Di era modern, board game tidak hanya menjadi sarana rekreasi, tetapi juga media edukasi dan sosial yang populer di kalangan berbagai usia. Menurut data dari [BoardGameGeek](https://boardgamegeek.com/) (BGG), terdapat lebih dari 19.000 board game yang telah dirilis dan diulas oleh komunitas pengguna secara global.

Namun, dengan jumlah board game yang begitu banyak dan beragam — baik dari segi tema, mekanik permainan, tingkat kompleksitas, hingga jumlah pemain — pengguna sering kali kesulitan menemukan game yang sesuai dengan preferensi mereka. Hal ini terutama dirasakan oleh pemain baru atau pengguna yang ingin menjelajahi genre baru namun tidak memiliki referensi yang memadai.

Dengan memanfaatkan pendekatan sistem rekomendasi, kita dapat membantu pengguna menemukan board game yang relevan dengan minat mereka. Sistem ini dapat bekerja berdasarkan konten game (seperti genre dan mekanik permainan) atau berdasarkan pola kesukaan pengguna lain yang serupa.

### **Kenapa Masalah Ini Harus Diselesaikan**

Membangun sistem rekomendasi board game dapat memberikan berbagai manfaat seperti:

- Membantu pengguna menemukan game yang sesuai preferensi tanpa harus menelusuri ribuan pilihan secara manual.
- Meningkatkan pengalaman pengguna, terutama bagi pemula yang masih mencari tahu game apa yang cocok untuk mereka.
- Memberikan nilai tambah bagi platform katalog game seperti BGG, toko board game online, atau aplikasi komunitas gamer.

Dengan meningkatnya tren personalisasi dan data-driven decision making, sistem rekomendasi menjadi solusi yang relevan dan aplikatif dalam membantu pengguna bernavigasi di tengah banyaknya pilihan board game yang tersedia.

### **Referensi Pendukung**

- A. Fanca, A. Puscasiu, D.-I. Gota and H. Valean, "Recommendation Systems with Machine Learning," 2020 21th International Carpathian Control Conference (ICCC), High Tatras, Slovakia, 2020, pp. 1-6, doi: 10.1109/ICCC49264.2020.9257290. [Link](https://ieeexplore.ieee.org/document/9257290)

---

## **Business Understanding**

### **Problem Statements**
- Bagaimana cara merekomendasikan board game yang sesuai dengan preferensi pengguna berdasarkan data atribut game seperti genre dan mekanik?

### **Goals**
- Membangun sistem rekomendasi board game dengan pendekatan Content-Based Filtering
- Mengevaluasi performa sistem rekomendasi untuk memastikan rekomendasi yang dihasilkan relevan.

### **Solution Statements**
- Menggunakan Content-Based Filtering dengan memanfaatkan atribut seperti Mechanics dan Domains untuk mengukur kemiripan antar board game.
- Melakukan evaluasi terhadap hasil rekomendasi menggunakan metrik Precision@K berdasarkan kesamaan konten.

---

## **Data Understanding**

Dataset yang digunakan adalah **Board Games Dataset** dari Kaggle: [Link Dataset](https://www.kaggle.com/datasets/melissamonfared/board-games). Dataset ini berisi informasi lebih dari 19.000 game dari situs BoardGameGeek.

### **Fitur-Fitur Utama:**

* **Board\_Game\_ID**: ID unik untuk setiap board game.
* **Name**: Nama dari board game.
* **Year\_Published**: Tahun pertama kali game tersebut dirilis.
* **Minimum\_Number\_of\_Players**: Jumlah pemain minimum yang direkomendasikan.
* **Maximum\_Number\_of\_Players**: Jumlah pemain maksimum yang dapat bermain.
* **Playing\_Time**: Estimasi waktu rata-rata bermain (dalam menit).
* **Recommended\_Minimum\_Age**: Usia minimum pemain yang direkomendasikan.
* **Number\_of\_User\_Ratings**: Jumlah total user yang telah memberikan rating terhadap game.
* **Average\_Rating**: Rata-rata nilai rating dari para pengguna.
* **BGG\_Rank**: Ranking board game berdasarkan komunitas BoardGameGeek.
* **Average\_Complexity**: Nilai rata-rata kompleksitas (semakin tinggi menunjukkan permainan yang lebih kompleks).
* **Number\_of\_BGG\_Registered\_Owners**: Jumlah pengguna yang mengaku memiliki board game tersebut.
* **Mechanics**: Mekanisme permainan yang digunakan dalam game (misalnya: dice rolling, area control).
* **Domains**: Kategori game berdasarkan jenis atau tema (misalnya: strategy, party, thematic).

### **Insight Penting:**

- Total data: 20.343 baris dengan 14 fitur utama.
- Tahun terbit rata-rata: 1984 (dengan outlier seperti tahun -3500).
- Jumlah pemain rata-rata: 2–5.
- Durasi bermain rata-rata: 91 menit (outlier: 60.000 menit dari *The Campaign for North Africa*).
- Rata-rata rating: 6.4, kompleksitas: 2.
- Korelasi tertinggi: `Users Rated` vs `Owned Users` (0.99).
- Korelasi terendah: `BGG Rank` vs `Average Rating` (-0.74).
- Lonjakan rilis board game signifikan sejak 2010-an.

---

## **Data Preparation**

Pada tahap Data Preparation, langkah-langkah yang dilakukan terbagi dalam 6 langkah untuk **Content-Based Filtering**

---

### **Untuk Content-Based Filtering:**

1. **Menghapus Data Kosong pada Kolom Kritis**

   * Baris dengan nilai kosong pada kolom `ID` dan `Year Published` dihapus.
   * Hal ini dilakukan untuk menghindari error saat analisis dan karena kolom tersebut penting dalam identifikasi item dan informasi temporal.

2. **Menangani Missing Values**

   * Kolom `Owned Users` diisi menggunakan **median** karena data bersifat skewed.
   * Kolom `Mechanics` dan `Domains` yang kosong diisi dengan `'unknown'` untuk tetap memungkinkan pemrosesan fitur teks.

3. **Filter Berdasarkan Popularitas**

   * Hanya board game dengan **lebih dari 50 user ratings** yang dipertahankan.
   * Tujuannya untuk menjamin kualitas data dan mengurangi bias dari game yang tidak populer.

4. **Transformasi Fitur Teks**

   * Kolom `Mechanics` dan `Domains` dikonversi menjadi **list of strings** yang telah dinormalisasi (lowercase dan strip).
   * Langkah ini penting sebagai bagian dari preprocessing sebelum digunakan dalam model berbasis konten.

5. **Seleksi Fitur**

   * Fitur yang dipertahankan:
     `Name`, `Rating Average`, `Complexity Average`, `Year Published`, `Min Players`, `Max Players`, `Play Time`, `Mechanics`, `Domains`
   * Fitur seperti `ID`, `BGG Rank`, dan `Users Rated` dihapus karena tidak relevan untuk proses pemodelan konten.

6. **Pembuatan Fitur Gabungan dan TF-IDF Vectorization**

   * Kolom `Mechanics` dan `Domains` digabungkan menjadi kolom baru `Combined_Features`.
   * Kemudian dilakukan proses **TF-IDF Vectorization** terhadap kolom ini untuk mengubah teks menjadi representasi numerik.
   * Hasil TF-IDF digunakan untuk menghitung **Cosine Similarity** antar game dalam model Content-Based Filtering.

---

## **Modeling & Result**

### **1. Content-Based Filtering**

**Konsep:**
- Rekomendasi diberikan berdasarkan kemiripan konten antar game, khususnya pada fitur `Mechanics` dan `Domains`.
- Game yang mirip akan dikenali dan direkomendasikan menggunakan pendekatan kemiripan teks.
- Sistem menghitung kesamaan antar game dan mengembalikan Top-N game yang paling mirip.

**Cara Kerja:**
- Setiap game direpresentasikan sebagai vektor berdasarkan kontennya.
- Kemiripan antar game dihitung, lalu sistem menyarankan game dengan konten paling mirip sebagai rekomendasi.

**Hasil:**
- Fungsi `recommend_games_content_based` pada `top-n` untuk `index 0`

|	|Name|Rating Average|Complexity Average|
|---|----|--------------|------------------|
|4504|Frosthaven|7.33|3.35|
|5|Gloomhaven: Jaws of the Lion|8.87|3.55|
|1218|Dragonfire|7.28|3.22|
|87|The Lord of the Rings: Journeys in Middle-Earth|8.08|2.57|
|21|Arkham Horror: The Card Game|8.17|3.44|
---


### **Kelebihan dan Kekurangan:**

| Metode | Kelebihan | Kekurangan |
|--------|-----------|------------|
| Content-Based | Tidak butuh data pengguna, bekerja dengan baik di cold-start | Kurang personal |

---

## **Evaluation**

### **1. Content-Based Filtering - Precision@K**

- Evaluasi: Precision@K dengan pendekatan kemiripan `Mechanics` dan `Domains`
- Hasil: 5 dari 5 rekomendasi memiliki kemiripan `Mechanics` dan `Domains` →  sangat baik

#### **Komparasi Hasil Evaluasi**
Evaluasi dilakukan menggunakan pendekatan **Content-Based Filtering (CBF)**, menggunakan metrik **Precision@5**. Berikut hasilnya:

| Metode               | Precision@5 | Keterangan                                                                 |
|----------------------|-------------|----------------------------------------------------------------------------|
| Content-Based        | **1.00**     | Semua rekomendasi sangat mirip secara `Mechanics` dan `Domains`.          |

Dari tabel di atas, dapat disimpulkan bahwa:
- **CBF unggul** dalam memberikan rekomendasi yang konsisten dan relevan berdasarkan konten game, terutama ketika data pengguna tidak tersedia.
- Evaluasi menunjukkan sistem dapat mengenali kemiripan konten dan menghasilkan rekomendasi yang sangat relevan.

---

### **2. Keterkaitan Hasil dengan Business Understanding**

- ✅ **Problem Statement Terjawab:** Sistem rekomendasi bekerja baik dengan pendekatan konten 
- 🎯 **Goals Tercapai:** Top-N rekomendasi relevan berhasil ditampilkan.
- 🧩 **Solusi Berdampak:** Content-Based: solusi tepat untuk cold-start user.

---

