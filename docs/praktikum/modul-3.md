# Dokumen Teknis Modul 3 — TypeScript dan Komponen React

**Nama/NIM:** Pradana Akbar Razan/105224005  
**Repositori:** https://github.com/PradanaAkbarRazan/prakweb-dokumen-teknis-modul3

## 1. Tipe Data Produk

Definisi tipe data terdapat pada `lib/tipe.ts`:

```ts
export type Kategori = "energi" | "lingkungan" | "kampus";

export interface Item {
  id: number;
  judul: string;
  kategori: Kategori;
  deskripsi: string;
  tersedia: boolean;
  gambar?: string;
}
```

`Kategori` menggunakan **union type** karena kategori item dibatasi pada tiga nilai yang diperbolehkan: `"energi"`, `"lingkungan"`, dan `"kampus"`. Dengan begitu, TypeScript dapat membantu mendeteksi penggunaan kategori di luar pilihan tersebut.

`Item` menggunakan `interface` untuk mendefinisikan struktur objek item. Properti `id`, `judul`, `kategori`, `deskripsi`, dan `tersedia` wajib diisi. Properti `gambar` bersifat opsional karena ditulis dengan tanda `?`; data tetap valid walaupun tidak menyertakan gambar.

Data contoh bertipe `Item[]` disimpan di `lib/data.ts`. Ada enam item: Panel surya atap, Bank sampah kampus, Kebun hidroponik, Lampu hemat energi, Peminjaman sepeda kampus, dan Tempat isi ulang air minum. Data untuk permintaan Fetch API tersedia pada `public/data/item.json`.

## 2. Struktur Komponen

### 2.1 Diagram pohon komponen

Diagram berikut menunjukkan hubungan layout dengan halaman, serta komponen yang digunakan pada masing-masing halaman. Halaman beranda dan katalog merupakan rute yang berbeda, sehingga keduanya tidak ditampilkan bersamaan pada satu halaman.

```text
RootLayout (Server Component, app/layout.tsx)
├── SiteHeader (navigasi bersama)
├── children (halaman aktif)
│   ├── Beranda — app/page.tsx (Server Component)
│   │   ├── Bagian — "Cara Kerja"
│   │   ├── Bagian — "Item Terbaru"
│   │   │   └── ItemTerbaru (Client Component)
│   │   └── Bagian — "Hubungi Kami"
│   └── HalamanKatalog — app/katalog/page.tsx (Server Component)
│       └── Bagian — "Daftar Item"
│           └── DaftarItem (Client Component)
│               └── KartuItem (dirender di dalam subtree client)
└── SiteFooter (footer bersama)
```

`RootLayout`, `Beranda`, dan `HalamanKatalog` tidak memakai direktif `"use client"`, sehingga menjadi Server Component secara bawaan pada Next.js App Router. `ItemTerbaru` dan `DaftarItem` memakai `"use client"` karena menggunakan React Hooks dan state. `KartuItem` diimpor oleh `DaftarItem`, sehingga digunakan sebagai bagian dari subtree Client Component pada katalog.

### 2.2 Tabel komponen

| Nama komponen | Props dan tipe | Tempat digunakan dan fungsi |
|---|---|---|
| `RootLayout` | `children` dari layout Next.js | `app/layout.tsx`; menyediakan kerangka HTML, tautan lewati ke konten utama, header, halaman aktif, dan footer. |
| `SiteHeader` | `namaProduk: string`; `tautan: { href: string; label: string }[]` | Di `RootLayout`; menampilkan nama produk dan tautan navigasi. |
| `SiteFooter` | `namaProduk: string` | Di `RootLayout`; menampilkan tahun dan nama produk. |
| `Beranda` | Tidak menerima props | `app/page.tsx`; menyusun pengenalan, cara kerja, item terbaru, dan formulir kontak. |
| `HalamanKatalog` | Tidak menerima props | `app/katalog/page.tsx`; menampilkan halaman katalog dan meneruskan `daftarItem` ke `DaftarItem`. |
| `Bagian` | `id: string`; `judul: string`; `children: React.ReactNode` | Dipakai pada beberapa bagian beranda dan katalog agar struktur section konsisten. |
| `ItemTerbaru` | Tidak menerima props | Di bagian “Item Terbaru” pada beranda; mengambil data dari `public/data/item.json`. |
| `DaftarItem` | `items: Item[]` | Di halaman katalog; menyediakan pencarian dan filter kategori. |
| `KartuItem` | `item: Item` | Dipakai oleh `DaftarItem` untuk menampilkan kategori, judul, deskripsi, dan status ketersediaan item. |

### 2.3 Alasan pemecahan komponen

`SiteHeader` dan `SiteFooter` ditempatkan pada layout agar dapat digunakan bersama di semua rute. `Bagian` mengurangi pengulangan struktur section. `DaftarItem` menangani interaksi pencarian dan filter, sementara `KartuItem` memisahkan tampilan satu item dari logika daftar. `ItemTerbaru` dipisahkan karena memiliki proses pengambilan data dan status pemuatan tersendiri.

Direktif `"use client"` hanya digunakan pada komponen yang membutuhkan interaksi atau Hooks. Pendekatan ini menjaga komponen halaman yang tidak memerlukan state interaktif sebagai Server Component.

## 3. State dan Interaksi

Di `components/DaftarItem.tsx`, terdapat dua state utama:

```tsx
const [kataKunci, setKataKunci] = useState("");
const [filter, setFilter] = useState<Filter>("semua");
```

`kataKunci` menyimpan teks pencarian, sedangkan `filter` menyimpan kategori yang dipilih. Tipe `Filter` didefinisikan sebagai `Kategori | "semua"`.

Alur interaksi hingga render ulang adalah sebagai berikut:

1. Saat pengguna mengetik pada kolom pencarian, event `onChange` memanggil `setKataKunci` dengan nilai input terbaru.
2. Saat pengguna menekan tombol kategori, event `onClick` memanggil `setFilter` dengan kategori yang dipilih.
3. Perubahan state membuat React merender ulang komponen.
4. Pada render berikutnya, variabel `hasil` dihitung dengan menyaring `items`: judul dicocokkan dengan kata kunci tanpa membedakan huruf besar-kecil, lalu kategori dicocokkan dengan filter aktif.
5. Jumlah hasil diperbarui. Jika tidak ada hasil, komponen menampilkan pesan “Tidak ada item yang sesuai.”; jika ada, hasil ditampilkan sebagai kartu.

Tombol filter menggunakan `aria-pressed` untuk menunjukkan filter aktif. Jumlah hasil menggunakan `aria-live="polite"` agar perubahan dapat diumumkan oleh teknologi bantu. Setiap item menggunakan `key={item.id}` agar React dapat mengenali elemen daftar secara stabil.

Bukti tab **React Developer Tools → Components**, yang memperlihatkan komponen `DaftarItem`, props `items`, serta state pencarian dan filter, tersedia pada **Lampiran A**.

## 4. Pengambilan Data

Komponen `ItemTerbaru` mengambil data dari `/data/item.json`, yang dilayani dari berkas `public/data/item.json`, menggunakan Fetch API di dalam `useEffect`.

Status pengambilan data menggunakan union type:

```ts
type Status = "memuat" | "berhasil" | "gagal";
```

Perilaku ketiga kondisi:

- **Memuat:** status awal adalah `"memuat"`; komponen menampilkan “Memuat data…”.
- **Berhasil:** Fetch API menerima respons, kode memeriksa `res.ok`, lalu membaca JSON sebagai `Item[]`. Tiga item terakhir dipilih dengan `data.slice(-3).reverse()` dan status diubah menjadi `"berhasil"`.
- **Gagal:** jika permintaan gagal atau `res.ok` bernilai `false`, status diubah menjadi `"gagal"` dan pesan “Data gagal dimuat. Muat ulang halaman.” ditampilkan dengan `role="alert"`.

Pemeriksaan `res.ok` penting karena Fetch API tidak otomatis melempar error hanya karena server merespons dengan status HTTP seperti 404. Fungsi cleanup di `useEffect` menggunakan penanda `dibatalkan` untuk mencegah pembaruan state setelah efek dibersihkan.

Bukti Network dan hasil pengujian dirangkum sebagai berikut:

| Kondisi | Hasil pengujian | Bukti |
|---|---|---|
| Memuat | Tulisan “Memuat data…” terlihat ketika permintaan `item.json` masih berlangsung. Pengujian dilakukan dengan throttling jaringan 3G dan Disable cache aktif. | Lampiran C |
| Berhasil | Permintaan `item.json` menampilkan Preview berisi enam item. Status yang tampak pada screenshot adalah `304` (resource tidak berubah dan dapat digunakan dari cache); screenshot pengujian loading juga memperlihatkan permintaan `200`. | Lampiran B |
| Gagal | Halaman menampilkan pesan “Data gagal dimuat. Muat ulang halaman.” saat pengambilan data gagal. URL pengujian yang salah harus dikembalikan ke `/data/item.json` setelah tes. | Lampiran D |

## 5. Verifikasi

Pemeriksaan dijalankan dari terminal proyek menggunakan perintah berikut:

```bash
npx tsc --noEmit
npm run lint
```

Pada tangkapan layar terminal, `npx tsc --noEmit` selesai dan kembali ke prompt tanpa menampilkan diagnostik. Perintah `npm run lint` menjalankan `eslint` dan juga kembali ke prompt tanpa menampilkan pesan error atau peringatan. Ini menunjukkan bahwa pada saat pengujian tersebut tidak ada diagnostik yang ditampilkan oleh kedua perintah.

Tangkapan layar terminal tersedia pada **Lampiran E**.

## 6. Kendala dan Penyelesaian

| Kendala yang ditemukan | Penanganan |
|---|---|
| Port 3000 sudah digunakan proses lain saat menjalankan `npm run dev`. | Next.js otomatis menjalankan aplikasi pada port 3001. Aplikasi kemudian diakses melalui alamat yang ditampilkan di terminal, misalnya `http://localhost:3001`. |
| Next.js menampilkan peringatan bahwa terdapat beberapa lockfile dan root workspace mungkin terdeteksi tidak tepat. | Peringatan tersebut dicatat saat menjalankan aplikasi. Peringatan tidak menghentikan server. Konfigurasi root workspace atau keberadaan lockfile perlu diperiksa jika ingin menghilangkan peringatan; hal ini tidak diklaim sudah diperbaiki dalam pengujian ini. |
| Saat menguji kondisi gagal, pengambilan data menghasilkan pesan kegagalan. | Setelah pengujian, pastikan URL Fetch API dikembalikan ke `/data/item.json`, lalu muat ulang halaman untuk memastikan data kembali dapat diambil. |

## 7. Catatan Pemanfaatan AI

**Alat:** ChatGPT.

**Bagian yang dibantu:** penyusunan kerangka dokumen teknis; penjelasan `Kategori`, `Item`, union type, dan properti opsional; penjelasan struktur Server dan Client Component; uraian alur state dan event; serta penjelasan Fetch API dan kondisi memuat, berhasil, dan gagal.

**Perintah atau kegiatan utama:** meminta bantuan untuk menyusun dokumen sesuai kerangka Bagian H Modul 3, memeriksa kesesuaian penjelasan dengan kode proyek di ZIP, serta mendapatkan panduan mengambil bukti dari React Developer Tools, Network, dan terminal.

**Cara verifikasi:** isi dokumen dibandingkan dengan kode proyek; perintah `npx tsc --noEmit` dan `npm run lint` dijalankan; pencarian/filter diperiksa melalui React Developer Tools; permintaan JSON diamati melalui Network; dan tangkapan layar pengujian disimpan sebagai bukti. Dokumen akhir tetap perlu diperiksa oleh penulis agar sesuai dengan kondisi repositori yang akan dikumpulkan.

---

# Lampiran Bukti Pengujian

Lampiran berikut mengumpulkan tangkapan layar di bagian akhir dokumen, sesuai pilihan format laporan.

## Lampiran A — React Developer Tools: `DaftarItem`

Screenshot menunjukkan komponen `DaftarItem`, props `items`, serta state untuk pencarian dan filter.

![React Developer Tools menampilkan DaftarItem, props, dan state](bukti/react%20DaftarItem%20week%203.png)

## Lampiran B — Network: respons `item.json`

Screenshot menunjukkan permintaan `item.json` dan Preview data JSON yang berisi enam item.

![Network dengan Preview respons item.json](bukti/Pengujian%20Kondisi%20Berhasil%20ItemJson%20week%203.png)

## Lampiran C — Pengujian kondisi memuat

Screenshot menunjukkan tulisan “Memuat data…” sementara permintaan `item.json` masih berlangsung dengan throttling 3G.

![Pengujian kondisi memuat dengan Network](bukti/Pengujian%20Kondisi%20Loading%20week%203.png)

## Lampiran D — Pengujian kondisi gagal

Screenshot menunjukkan pesan kegagalan yang ditampilkan komponen ketika data tidak berhasil dimuat.

![Pengujian kondisi gagal](bukti/Pengujian%20kondisi%20gagal%20week%203.png)

## Lampiran E — Verifikasi TypeScript dan ESLint

Screenshot terminal menunjukkan pelaksanaan `npx tsc --noEmit` dan `npm run lint`.

![Hasil pemeriksaan TypeScript dan ESLint](bukti/tes%20npx%20tsc%20--noEmit%20dan%20npm%20run%20lint%20week%203.png)
