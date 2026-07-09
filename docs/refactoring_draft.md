# Draft: Modernization & Refactoring Strategy for SilentXMRMiner

This document outlines the proposed strategies for modernizing and refactoring the SilentXMRMiner codebase. The primary goals are to improve maintainability, separate concerns, and update the technology stack to modern standards, strictly for educational and ethical software engineering purposes.

## 1. Pemisahan Logika (Separation of Concerns / Clean Architecture) - PRIORITAS UTAMA
*   **Tujuan:** Memisahkan UI (Windows Forms) dari logika bisnis/kompilasi agar kode lebih mudah dikelola dan diuji.
*   **Analisis Saat Ini (`Codedom.vb`):**
    *   File ini bertindak sebagai "Builder Engine" utama.
    *   Ketergantungan UI sangat tinggi. Contoh pada fungsi `ReplaceGlobals`:
        *   Baris 242: `If F.FA.toggleKillWD.Checked Then` (Membaca status checkbox UI secara langsung).
        *   Baris 260: `Select Case F.txtInstallPathMain.Text` (Membaca teks dari textbox UI).
        *   Baris 300: `stringb.Replace("%Title%", F.txtTitle.Text)` (Mengambil nilai UI untuk mengganti variabel placeholder).
        *   Objek `F` tampaknya merupakan referensi statis atau global ke `Form1` (form utama UI).
*   **Strategi Refactoring (`Codedom.vb` & Form UI):**
    1.  **Buat Kelas Konfigurasi (Model):** Buat sebuah kelas baru (misalnya `BuilderConfig.vb`) yang berisi properti murni untuk semua opsi *builder*. Properti ini bertipe data standar (Boolean, String, Integer) tanpa referensi ke UI.
        *   Contoh: `Public Property IsKillWdEnabled As Boolean`, `Public Property InstallPath As String`.
    2.  **Ubah Parameter Fungsi:** Ubah fungsi `Compiler`, `UninstallerCompiler`, dan `ReplaceGlobals` di `Codedom.vb` agar menerima objek `BuilderConfig` sebagai parameter, bukan mengakses variabel UI `F` secara langsung.
    3.  **Mapping di UI:** Pada tombol "Build" di UI, buat *instance* dari `BuilderConfig`, isi nilainya berdasarkan status kontrol UI saat itu, lalu teruskan ke fungsi-fungsi di `Codedom.vb`.

## 2. Peningkatan Kualitas dan Keterbacaan Kode (Clean Code)
*   **Refactoring Variabel & Fungsi:** Mengubah nama agar lebih deskriptif (misalnya `F.FA` menjadi nama yang lebih jelas seperti `AdvancedSettingsForm`).
*   **Penanganan Error (Error Handling):** Memperbaiki blok `Try-Catch` (seperti di baris 199 `Codedom.vb`) agar tidak sekadar memunculkan `MessageBox` tetapi juga mengembalikan status sukses/gagal ke pemanggilnya, sehingga UI bisa merespons dengan tepat.
*   **Penghapusan "Magic Strings":** Memindahkan *string* hardcode (seperti nama folder atau perintah shell) ke konstanta.

## 3. Penambahan Pengujian Otomatis (Unit Testing)
*   **Tujuan:** Memastikan logika *builder* bekerja dengan benar setelah UI dipisahkan.
*   **Langkah:**
    *   Buat proyek Unit Test (`SilentXMRMiner.Tests`).
    *   Buat *test cases* untuk `ReplaceGlobals` dengan mensimulasikan berbagai kondisi di `BuilderConfig` dan memeriksa apakah `StringBuilder` dimodifikasi dengan benar (misalnya `DefGPU` disisipkan jika `IsGpuEnabled` = True).
    *   Buat tes untuk fungsi utilitas seperti `F.Cipher` dan generator *random string*.

## 4. Migrasi Framework (Pembaruan Teknologi Utama) - JANGKA PANJANG
*   **Dari .NET Framework (lama) ke .NET 8:**
    *   Mengubah format proyek menjadi SDK-style (C# / VB.NET).
    *   *Tantangan Spesifik:* Penggunaan `CSharpCodeProvider` (baris 210 `Codedom.vb`) untuk mengkompilasi file *uninstaller* dan metode kompilasi payload menggunakan `tcc`/`windres` secara dinamis. Di .NET modern, kompilasi *runtime* seringkali lebih baik menggunakan `Microsoft.CodeAnalysis.CSharp` (Roslyn) daripada `System.CodeDom`. Ini akan memerlukan penulisan ulang bagian kompilasi C#.

## 5. Migrasi Bahasa (Opsional)
*   Menerjemahkan *codebase* dari **VB.NET** ke **C#** secara bertahap setelah logika UI dan *business logic* terpisah dengan baik.

---
**Rencana Tindakan (Action Plan):**
Langkah pertama yang paling krusial dan memiliki dampak paling besar terhadap kebersihan kode adalah **Langkah 1 (Pemisahan Logika)**. Ini akan memutus ketergantungan antara mesin *builder* dan UI, membuka jalan untuk pengujian otomatis dan migrasi *framework* yang lebih mudah.
