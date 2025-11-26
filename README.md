# Image-Stiching
Rekontruksi dari project Computer Vision (mata kuliah semester 5)

# Image Stitching Pipeline (Szeliski-Inspired)

Project ini mengimplementasikan proses *image stitching* berdasarkan alur yang direkomendasikan Richard Szeliski:
**preprocessing → feature detection → feature matching → RANSAC homography → warping → blending**.

Dua algoritma feature extraction dievaluasi: **SIFT** dan **ORB**.

---

## Tujuan
- Mengimplementasikan panorama stitching end-to-end menggunakan OpenCV.
- Membandingkan performa SIFT dan ORB pada gambar yang tidak seimbang (exposure berbeda, overlap kecil, distorsi, dll).
- Memberikan visualisasi setiap tahap: keypoints, matches, homography inliers, warping, dan panorama final.

---

## Dataset
Menggunakan dua foto overlap yang diambil manual, dengan kondisi:
- Overlap tidak stabil
- Exposure tidak seragam
- Terdapat area low texture (langit/dinding)
- Ada noise dari pergerakan kamera

Kondisi ini dibuat untuk menguji robustness SIFT & ORB.

---

## Metode
### 1. Preprocessing
- Resizing 50%
- Grayscale & histogram equalization

### 2. Fitur
- **SIFT** (Scale-Invariant Feature Transform)  
  + float descriptor 128D  
  + tahan skala, rotasi, iluminasi  
  + lebih stabil pada kondisi real-world yang tidak ideal  

- **ORB** (Oriented FAST + BRIEF)  
  + binary descriptor  
  + cepat dan ringan  
  + sensitif pada perbedaan exposure & low-texture  

### 3. Matching
- SIFT → BFMatcher + L2 + Lowe ratio test  
- ORB → BFMatcher + Hamming + ratio test  

### 4. Homography Estimation
- Menggunakan RANSAC
- Mengambil inlier → menentukan transformasi perspektif

### 5. Warping & Blending
- `warpPerspective` untuk menempatkan citra ke koordinat panorama  
- Distance-based feather blending untuk mengurangi seam  

---

## Hasil & Analisis

### **Kesimpulan Utama**
**SIFT menghasilkan panorama yang lebih stabil, konsisten, dan robust dibandingkan ORB pada gambar yang tidak seimbang.**

### Kenapa SIFT lebih baik?
- Menemukan **lebih banyak keypoints yang relevan** pada area tekstur rendah.
- Descriptor float 128D **lebih informatif**, sehingga matching lebih akurat.
- Lebih tahan terhadap:
  - exposure difference  
  - blur kecil  
  - pergeseran kecil kamera  
  - overlap yang tidak simetris  

### Kenapa ORB kurang stabil?
- ORB bergantung pada FAST corner detection → gagal pada area smooth.
- BRIEF descriptor sensitif terhadap perubahan intensitas.
- Jumlah keypoints efektif rendah → RANSAC sering gagal menemukan homography yang stabil.
- Panorama cenderung:
  - misaligned  
  - terdistorsi  
  - menghasilkan seam terlihat  

### Visual Overview
- SIFT → matching lebih rapat & inlier RANSAC lebih banyak  
- ORB → banyak false match, inlier sedikit, hasil warped bergeser

---

## Performance Comparison

| Criteria                      | SIFT | ORB |
|------------------------------|------|-----|
| Robustness pada exposure     | ⭐⭐⭐⭐ | ⭐⭐ |
| Keypoints pada low-texture   | ⭐⭐⭐⭐ | ⭐ |
| Match accuracy               | ⭐⭐⭐⭐ | ⭐⭐ |
| RANSAC inliers               | Tinggi | Rendah |
| Processing speed             | Lebih lambat | Sangat cepat |
| Panorama final               | Stabil | Sering distorsi |

---

## Kesimpulan Akhir
**SIFT adalah pilihan terbaik untuk image stitching pada kondisi foto yang sulit**, walaupun lebih lambat.  
Jika environment terbatas (mobile, real-time, embedded), ORB bisa dipakai — tetapi kualitas panorama bukan prioritas.

---

## Rekomendasi Selanjutnya
- Tambahkan cylindrical projection untuk panorama yang lebih lebar.
- Implementasikan multiband blending (Laplacian Pyramid).
- Evaluasi algoritma lain: AKAZE, SuperPoint, LoFTR.
- Test stitching untuk 3–5 gambar sekaligus.

---

## Dependencies
- OpenCV (opencv-contrib-python wajib untuk SIFT)
- NumPy
- Matplotlib
- SciPy

