# Instruksi Konversi: Dokumen Internal IT → md FRS

Dokumen ini adalah instruksi untuk AI agent. Berikan file ini ke agent bersama dua file lain:

1. **Dokumen sumber** — dokumen internal IT yang sudah final, ditulis mengikuti `template-internal-it.md`. Biasanya satu; jika beberapa dokumen harus menjadi satu FRS, ikuti juga bagian *Jika dokumen sumber lebih dari satu*.
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

## Jika dokumen sumber lebih dari satu

Bagian ini berlaku jika System Analyst memberikan beberapa dokumen internal IT dan meminta **satu** FRS untuk semuanya. Jika beberapa dokumen diberikan tanpa permintaan itu, hasilkan satu FRS per dokumen seperti biasa.

Semua aturan di atas tetap berlaku untuk tiap dokumen. Bagian ini mengatur cara menyatukan hasilnya, dan menggantikan aturan di atas hanya di tempat yang disebut. Di bawah ini, "pekerjaan" berarti isi satu dokumen sumber, dan namanya adalah *Judul FRS* di *Identitas FRS* dokumen itu.

### Masukan dan keluaran

- **Judul dan nomor FRS gabungan diberikan System Analyst.** Keduanya tidak ada di dokumen sumber mana pun. Jika tidak diberikan, tulis `{{PERLU DIISI: judul FRS gabungan}}` dan `{{PERLU DIISI: nomor FRS}}` di front matter dan di judul `#`. Jangan merangkai judul dari judul-judul pekerjaan.
- **Nama file** (menggantikan aturan di *Keluaran*): `frs-<judul gabungan, huruf kecil, kata dipisah tanda hubung>.md`, atau `frs-gabungan.md` jika judulnya belum ada. Disimpan di folder dokumen sumber pertama.
- **Urutan pekerjaan** mengikuti urutan yang diberikan System Analyst. Jika tidak disebut dan nama file diawali angka (`1. ...`, `2. ...`), pakai angka itu sebagai urutan, dibaca sebagai bilangan sehingga `10` jatuh setelah `9`, bukan setelah `1`. Jika tidak ada angka, pakai urutan abjad nama file. Urutan ini dipakai di semua bagian FRS, dan ditulis di laporan supaya bisa diperiksa.
- `created_by` ← `author`; jika berbeda antar dokumen, tulis semuanya dipisah koma. *Bahasa FRS* seharusnya sama di semua dokumen; jika berbeda, pakai bahasa dokumen pertama dan laporkan.

### Pemeriksaan final

Pemeriksaan di *Sebelum mulai* dijalankan untuk tiap dokumen. Satu dokumen yang belum final membuat seluruh FRS gabungan belum final, karena FRS ditandatangani sebagai satu kesatuan. Laporkan kondisinya per dokumen.

Dokumen yang menyatakan isinya bergantung pada keputusan di dokumen lain diperiksa bersama dokumen itu: jika keputusan yang dirujuk masih terbuka, keduanya belum final.

### Aturan penggabungan

**Isi tiap pekerjaan hanya diambil dari dokumennya sendiri.** Validasi, asumsi, atau perilaku yang tertulis di satu dokumen tidak diterapkan ke halaman milik dokumen lain, walaupun halamannya mirip.

**Rujukan internal dokumen tidak ikut ke FRS.** "Flow 1", "Halaman 2", "temuan 5", dan nama file sumber hanya bermakna di dalam dokumennya, dan nomor yang sama menunjuk hal berbeda di dokumen lain. Di FRS, sebut nama proses atau nama halamannya. Rujukan seperti itu hanya boleh muncul di dalam `{{PERLU DIISI: ...}}`, karena placeholder dibaca System Analyst.

**Yang sama ditulis sekali; yang bertentangan tidak diputuskan sendiri.** Saat dua dokumen menyebut hal yang sama (role, istilah, asumsi, field), bedakan tiga keadaan:

- *Maknanya sama, rumusannya berbeda* — tulis sekali. Pilih rumusan yang benar untuk semua pekerjaan di FRS, tanpa merangkai rumusan baru, dan cantumkan pilihan itu di laporan.
- *Saling melengkapi* (misalnya dua dokumen mengubah field berbeda di halaman yang sama) — gabungkan.
- *Bertentangan* — tulis `{{PERLU DIISI: <kedua pernyataan dan dokumen asalnya>}}` di tempatnya dan cantumkan di laporan. Memilih salah satu berarti memutuskan requirement atas nama System Analyst.

Cara menggabungkan per bagian:

| Bagian FRS | Cara menggabungkan |
|---|---|
| 1.1 OBJECTIVES | Numbered list, satu butir per pekerjaan. Jangan menulis tujuan payung yang tidak tertulis di dokumen mana pun. |
| 1.2 BACKGROUND | Satu paragraf pendek per pekerjaan, diawali nama pekerjaan dalam huruf tebal. |
| 1.3 SCOPE | Paragraf ringkasan menyebut semua pekerjaan. Satu tabel untuk semuanya: baris diurutkan per pekerjaan dan kolom *No* dinomori ulang dari 1. Jika FRS mencakup lebih dari satu aplikasi, kolom *Modul* diawali nama aplikasinya. Kalimat *Tidak termasuk* dan rekomendasi yang belum masuk scope digabung, dengan satu pengecualian: butir yang dikecualikan satu dokumen tetapi dikerjakan dokumen lain di FRS yang sama tidak boleh ditulis sebagai tidak termasuk. Hapus butir itu jika dokumen lain mencakupnya seluruhnya; jika hanya sebagian, tulis dengan batasnya (apa yang termasuk dan apa yang tidak). Cantumkan di laporan. |
| 1.4 WORKFLOW | *Aturan Workflow* diterapkan per pekerjaan, dan kalimat pengantar tiap diagram menyebut nama pekerjaannya. Flow dari dokumen berbeda tidak digabung menjadi satu diagram, kecuali dokumen sumber sendiri menyatakan flow itu lanjutan dari flow di dokumen lain. `**Notes:**` tetap satu, di bawah diagram terakhir, dan tiap butirnya menyebut proses yang dimaksud. |
| 1.5 ASSUMPTIONS | Satu numbered list, urut per pekerjaan. Asumsi yang hanya berlaku untuk satu pekerjaan dan menjadi ambigu di luar dokumennya diberi konteks halaman atau prosesnya. |
| 1.6 USER | Satu baris per role per aplikasi. Role yang sama dari beberapa dokumen dilebur menjadi satu baris, dan deskripsinya merangkum apa yang dilakukan role itu di semua pekerjaan. Role bernama sama di aplikasi berbeda ditulis terpisah, dengan nama aplikasi di kolom *User / Role*. |
| 1.7 TERMINOLOGY | Satu baris per istilah. Istilah yang artinya memang berbeda per aplikasi ditulis terpisah dengan nama aplikasinya. |
| 2.1 ringkasan | Satu paragraf pembuka, lalu bullet list satu butir per pekerjaan. |
| 2.2, 2.3 | Tabel digabung dan *No* dinomori ulang. `N/A` hanya jika semua dokumen menulis N/A. |
| 2.4 | Satu numbered list; tiap butir menyebut halaman yang dimaksud. `N/A` hanya jika semua dokumen menulis N/A. |

Untuk blok `2.1.N`:

- Semua blok *Halaman N* dari semua dokumen ikut, diurutkan per pekerjaan lalu per urutan halaman di dokumennya, dan dinomori berurutan mulai dari `2.1.1`.
- *Application Name* diambil dari *Identitas FRS* dokumen asal blok itu. Ini menggantikan aturan "sama untuk semua blok": satu FRS gabungan bisa mencakup lebih dari satu aplikasi.
- Dua blok dilebur menjadi satu hanya jika *Application Name*, *Page Name*, *Component Name*, dan *Role*-nya sama. Blok hasil leburan ditempatkan di posisi kemunculan pertamanya. Tabel *Fields* digabung; field yang sama ditulis satu baris dengan *Details / Validation* dari kedua dokumen. *New/Existing* ditulis `New` jika salah satu dokumen menyebut halaman itu baru.
- Jika *Page Name* sama tetapi *Component Name* atau *Role* berbeda, bloknya tetap terpisah.
- Halaman yang sama ditulis dengan *Navigation* berbeda di dua dokumen: pakai yang muncul pertama dan cantumkan di laporan.
- Pemeriksaan *Validasi & Aturan Bisnis* memakai flow di dokumen asal blok itu. Untuk blok hasil leburan, periksa flow terkait di tiap dokumen asalnya.

## Pemeriksaan sebelum menyerahkan

- Semua heading bernomor dari template ada, dengan urutan dan penulisan yang sama.
- Tiap blok `2.1.N` punya empat sub-heading dalam urutan yang benar.
- Header kolom setiap tabel sama persis dengan template.
- Tidak ada `{{...}}` selain `{{PERLU DIISI: ...}}`.
- Tidak ada nama tabel, kolom, endpoint, atau istilah teknis yang terbawa. Cari backtick di keluaran: hampir semua backtick menandakan istilah teknis yang lolos.
- Setiap blok `mermaid` adalah `flowchart` dengan sintaks yang valid.

Tambahan untuk FRS gabungan:

- Setiap baris *Scope* dan setiap blok *Halaman N* dari setiap dokumen sumber ada di FRS, berdiri sendiri atau terlebur. Hitung per dokumen; yang paling mudah hilang saat menggabungkan adalah baris dari dokumen di tengah urutan.
- Kolom *No* di tiap tabel berurutan tanpa lompatan dan tanpa nomor ganda.
- Tidak ada role atau istilah yang muncul dua kali untuk aplikasi yang sama.
- Tidak ada "Flow N", "Halaman N", nomor temuan, atau nama file sumber di luar `{{PERLU DIISI: ...}}`.
- Tidak ada butir di kalimat *Tidak termasuk* yang bertentangan dengan baris di tabel *Scope*.

## Format laporan konversi

Tulis singkat, dalam urutan ini:

1. **File yang dihasilkan** — nama file.
2. **Perlu diisi** — daftar tiap `{{PERLU DIISI: ...}}` beserta section-nya. Tulis "tidak ada" jika kosong. Tambahkan satu baris pengingat bahwa Running ID, Application ID, Hierarchy ID - Name, dan screenshot Screen Layout tiap blok `2.1.N` harus diisi manual di Word.
3. **Yang ditambahkan dari luar *Bahan FRS*** — validasi yang dipindahkan ke *Fields Detail*, istilah yang ditambahkan ke *Terminology*, dan sejenisnya, beserta asal section-nya, supaya System Analyst bisa memeriksanya.
4. **Yang perlu diputuskan System Analyst** — hal di dokumen sumber yang ambigu atau saling bertentangan dan memengaruhi isi FRS. Sebutkan pilihan yang Anda ambil.

Untuk FRS gabungan, tiap butir di nomor 2 sampai 4 menyebut dokumen asalnya, status final dilaporkan per dokumen, dan laporan ditambah satu bagian:

5. **Hasil penggabungan** — blok `2.1.N` yang dilebur; role, istilah, dan asumsi yang dilebur beserta rumusan yang dipilih; butir *Tidak termasuk* atau rekomendasi yang dihapus atau dipersempit karena dikerjakan dokumen lain; dan pertentangan antar dokumen yang ditulis sebagai `{{PERLU DIISI: ...}}`.
