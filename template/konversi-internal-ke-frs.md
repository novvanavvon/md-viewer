# Instruksi Konversi: Dokumen Internal IT → md FRS

Dokumen ini adalah instruksi untuk AI agent. Berikan file ini ke agent bersama dua file lain:

1. **Dokumen sumber** — dokumen internal IT yang sudah final, ditulis mengikuti `template-internal-it.md`.
2. **`template-frs.md`** — bentuk keluaran yang harus diikuti.

## Konteks

Dokumen internal IT ditulis System Analyst untuk programmer, QA, dan DBA. Isinya teknis: tabel, kolom, query, transaksi, endpoint.

FRS (Functional Requirement Statement) adalah dokumen resmi yang dibaca dan ditandatangani user, requester, dan manajer. Mereka tidak membaca kode dan tidak perlu tahu nama tabel. Mereka perlu tahu apa yang berubah bagi mereka, halaman apa yang mereka lihat, dan field apa yang mereka isi.

Tugas Anda adalah menulis ulang isi dokumen sumber menjadi md FRS untuk pembaca tersebut. Ini pekerjaan menerjemahkan dan memilih, bukan menyalin: sebagian besar isi teknis tidak ikut, dan sisanya ditulis ulang dengan bahasa bisnis.

md FRS yang Anda hasilkan nantinya dibaca program yang mengisinya ke template Word. Program itu mencari heading dan kolom tabel berdasarkan nama persisnya, sehingga struktur keluaran tidak boleh menyimpang dari `template-frs.md`.

## Keluaran

Hasilkan dua hal:

1. **Satu file md FRS**, disimpan di folder yang sama dengan dokumen sumber, dengan nama `frs-<nama-file-sumber>.md`.
2. **Laporan konversi** sebagai balasan ke System Analyst (format di bagian akhir). Laporan tidak ditulis ke dalam file FRS.

Dokumen sumber tidak boleh diubah.

## Sebelum mulai: periksa apakah dokumen sumber sudah final

Hentikan konversi dan laporkan ke System Analyst jika salah satu kondisi ini ditemukan, karena FRS dari dokumen yang belum final akan ditandatangani dengan isi yang masih bisa berubah:

- Masih ada `{{...}}` di luar komentar HTML.
- Masih ada butir di *Perlu Didiskusikan* yang berstatus belum ditetapkan.
- Masih ada isi contoh bawaan template (`TrContoh`, `MsContohInduk`, `TrContohLog`, `/api/contoh`).
- *Bagian 3 — Bahan FRS* tidak ada.

Jika System Analyst meminta konversi tetap dilanjutkan sebagai draft, lanjutkan, dan tulis kondisi di atas di laporan.

## Aturan

**Struktur mengikuti `template-frs.md` persis.** Heading bernomor (`## 1. INTRODUCTION` sampai `### 2.4 INTERFACE REQUIREMENT`), nama dan urutan kolom tabel, serta empat sub-heading di tiap blok `2.1.N` tidak diubah, tidak dihapus, dan tidak ditambah. Section yang memang tidak relevan tetap ditulis dengan isi `N/A`.

**Hanya tulis yang ada di dokumen sumber.** Jika informasi untuk satu bagian tidak ada, tulis `{{PERLU DIISI: <apa yang kurang>}}` di tempatnya dan cantumkan di laporan. Jangan mengisi dengan perkiraan yang masuk akal: FRS ditandatangani, dan isi karangan yang terlihat meyakinkan lebih berbahaya daripada bagian yang jelas-jelas kosong.

**Gunakan bahasa bisnis.** Tidak boleh muncul di FRS: nama tabel dan kolom, SQL, nama class/method/job, endpoint dan kode HTTP, istilah transaksi dan locking, nomor keputusan desain (`D1`, `D2`), dan nomor `Seq`. Tulis akibatnya bagi user, bukan mekanismenya. Contoh:

| Di dokumen sumber | Di FRS |
|---|---|
| "INSERT `TrContoh` dengan `Status = 'SUBMITTED'`" | "Sistem menyimpan pengajuan dengan status Submitted." |
| "API mengembalikan 409 jika unique index dilanggar" | "Sistem menolak pengajuan kedua untuk data yang sama dan menampilkan pesan." |
| "Retry 3x dengan exponential backoff" | "Jika pengiriman gagal, sistem mencoba mengirim ulang secara otomatis hingga 3 kali." |

Nama status yang terlihat user di layar boleh dipakai apa adanya.

**Bahasa isi** mengikuti baris *Bahasa FRS* di *Identitas FRS*. Heading dan nama kolom tabel tetap dalam bahasa Inggris seperti di template, apa pun bahasa isinya.

**Elemen yang boleh dipakai** hanya paragraf, bullet/numbered list, tabel, blok `mermaid`, dan gambar `![caption](path)`. Tanpa HTML mentah, tanpa tabel di dalam tabel, tanpa heading tambahan. Komentar HTML petunjuk dari `template-frs.md` tidak perlu disalin ke keluaran.

## Pemetaan

| Bagian FRS | Sumber di dokumen internal | Cara menulis |
|---|---|---|
| Front matter | *Identitas FRS* | `doc_type` tetap `FRS`. `frs_number` ← Nomor FRS, `title` ← Judul FRS, `created_by` ← `author`. `division` dan `document_version` ikut nilai template. `document_date` ← tanggal konversi. `title` dan `frs_number` wajib terisi karena program memakainya untuk mengisi judul dan nomor di cover dan header Word; tanggal di Word diisi program dengan tanggal saat export. |
| Judul `#` | *Identitas FRS* | `# <Nomor FRS> - <Judul FRS>` |
| 1.1 OBJECTIVES | Paragraf pembuka (bagian solusi), *Gambaran Besar* | Tujuan dari sudut pandang user: apa yang bisa mereka lakukan atau apa yang membaik. Bukan tujuan teknis. |
| 1.2 BACKGROUND | Paragraf pembuka (bagian masalah), *Temuan* | Kondisi saat ini dan masalahnya. Dari *Temuan*, ambil hanya yang dirasakan user; temuan soal index, timeout, atau struktur kode tidak ikut. |
| 1.3 SCOPE | *Bahan FRS → Scope*, *Rekomendasi (Belum Masuk Scope)* | Paragraf dan tabel disalin. Baris *Tidak termasuk* dan butir rekomendasi yang belum masuk scope ditulis sebagai kalimat di paragraf. |
| 1.4 WORKFLOW | *Activity Diagram* tiap flow | Lihat *Aturan Workflow* di bawah. |
| 1.5 ASSUMPTIONS | *Bahan FRS → Asumsi* | Disalin sebagai numbered list. |
| 1.6 USER | *Aktor & Hak Akses* | Satu baris per role. Kolom *Boleh* dan *Tidak boleh* dirangkum menjadi satu deskripsi dalam bahasa user. Aktor non-manusia (scheduler, job) tidak ditulis sebagai user. |
| 1.7 TERMINOLOGY | *Bahan FRS → Istilah* | Disalin. Tambahkan istilah lain hanya jika istilah itu Anda pakai di FRS dan artinya tertulis di dokumen sumber. |
| 2.1 ringkasan | *Gambaran Besar*, tabel ringkasan tiap flow (baris *Tujuan* dan *Hasil akhir*) | Satu sampai dua paragraf tentang apa yang dilakukan sistem, tanpa nama komponen teknis. |
| 2.1.N | *Bahan FRS → Halaman & Field* | Lihat *Aturan blok 2.1.N* di bawah. |
| 2.2 DATA REQUIREMENT | *Bahan FRS → Kebutuhan Data* | Disalin. |
| 2.3 REPORTING REQUIREMENT | *Bahan FRS → Kebutuhan Laporan* | Disalin. |
| 2.4 INTERFACE REQUIREMENT | *Bahan FRS → Kebutuhan Interface* | Disalin. |

Bagian berikut tidak ikut ke FRS dalam bentuk apa pun: *Keputusan Desain & Alasannya*, *DMD* (seluruhnya), *Konfigurasi*, *Sequence Diagram*, *CRUD / Akses Data*, *Query*, *Transaksi & Konkurensi*, *Kontrak API*, *Penyesuaian di Aplikasi*, dan *Rencana Deploy & Testing*.

### Aturan Workflow

Workflow di FRS adalah alur bisnis: siapa melakukan apa, dan apa tanggapan sistem.

- Satu diagram `flowchart TD` per flow yang melibatkan user. Jika beberapa flow membentuk satu proses bisnis yang berurutan, gabungkan menjadi satu diagram.
- Beri satu kalimat pengantar sebelum tiap diagram.
- Tiap node adalah aktivitas aktor atau sistem yang terlihat oleh user. Langkah yang hanya terjadi di dalam sistem (baca tabel, transaksi, commit, rollback) digabung menjadi satu node seperti "Sistem menyimpan pengajuan".
- Cabang keputusan yang ikut hanya yang hasilnya terlihat user, misalnya data tidak valid atau tidak berhak.
- Pertahankan gaya sintaks template: label di dalam tanda kutip, bentuk `([..])` untuk awal/akhir, `{..}` untuk keputusan. Jangan memakai `subgraph`, styling, atau `click`; diagram ini akan diubah menjadi gambar dan bentuk sederhana paling aman.
- Di bawah diagram terakhir tulis `**Notes:**` berisi penjelasan yang tidak muat di diagram, diambil dari *Alur Alternatif & Kegagalan* yang relevan bagi user. Jika tidak ada, tulis `N/A`.

Flow yang seluruhnya berjalan tanpa user (scheduler, job) tetap digambarkan jika hasilnya sampai ke user, misalnya email yang mereka terima. Tulis dari sisi yang terlihat: kapan dijalankan dan apa yang diterima user.

### Aturan blok 2.1.N

- Satu blok *Halaman N* di *Bahan FRS* menjadi satu blok `#### 2.1.N <Application Name> - <Role>`, dengan urutan yang sama dan nomor berurutan mulai dari `2.1.1`.
- **Detail Information**: hanya *Application Name* yang diisi, dari *Identitas FRS*, dan sama untuk semua blok. Baris *Running ID*, *Application ID*, dan *Hierarchy ID - Name* tetap ditulis tetapi nilainya dikosongkan (sel kosong, tanpa `N/A` dan tanpa `{{PERLU DIISI}}`), walaupun nilainya disebut di dokumen sumber: ketiganya diisi manual oleh System Analyst di dokumen Word.
- **Page Detail** diisi dari tabel halaman tersebut. *New/Existing* ditulis persis `New` atau `Existing`. Baris *Role* dan *Flow terkait* tidak ikut ke tabel ini.
- **Screen Layout**: heading-nya tetap ditulis, isinya dikosongkan. Screenshot dimasukkan manual oleh System Analyst di dokumen Word, jadi jangan menulis `{{PERLU DIISI}}`, jangan membuat gambar, dan jangan menggantinya dengan deskripsi teks. Hanya jika dokumen sumber sudah memuat baris gambar `![...](...)`, salin baris itu apa adanya tanpa mengubah path.
- **Fields Detail**: salin tabel *Fields*. *Given / Input* ditulis persis `Given` atau `Input`, *Mandatory* persis `Yes` atau `No`.
- Periksa *Validasi & Aturan Bisnis* pada flow yang tercantum di *Flow terkait*. Aturan yang menyangkut suatu field tetapi belum tertulis di kolom *Details / Validation* field itu ditambahkan ke sana dalam bahasa user, dan dicantumkan di laporan.

## Pemeriksaan sebelum menyerahkan

- Semua heading bernomor dari template ada, dengan urutan dan penulisan yang sama.
- Tiap blok `2.1.N` punya empat sub-heading dalam urutan yang benar.
- Header kolom setiap tabel sama persis dengan template.
- Tidak ada `{{...}}` selain `{{PERLU DIISI: ...}}`.
- Tidak ada nama tabel, kolom, endpoint, atau istilah teknis yang terbawa. Cari backtick di keluaran: hampir semua backtick menandakan istilah teknis yang lolos.
- Setiap blok `mermaid` adalah `flowchart` dengan sintaks yang valid.

## Format laporan konversi

Tulis singkat, dalam urutan ini:

1. **File yang dihasilkan** — nama file.
2. **Perlu diisi** — daftar tiap `{{PERLU DIISI: ...}}` beserta section-nya. Tulis "tidak ada" jika kosong. Tambahkan satu baris pengingat bahwa Running ID, Application ID, Hierarchy ID - Name, dan screenshot Screen Layout tiap blok `2.1.N` harus diisi manual di Word.
3. **Yang ditambahkan dari luar *Bahan FRS*** — validasi yang dipindahkan ke *Fields Detail*, istilah yang ditambahkan ke *Terminology*, dan sejenisnya, beserta asal section-nya, supaya System Analyst bisa memeriksanya.
4. **Yang perlu diputuskan System Analyst** — hal di dokumen sumber yang ambigu atau saling bertentangan dan memengaruhi isi FRS. Sebutkan pilihan yang Anda ambil.
