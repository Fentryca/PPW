# Crawlling Berita dari detik.com

```python
!pip install builtwith
```

    Collecting builtwith
      Downloading builtwith-1.3.4.tar.gz (34 kB)
      Preparing metadata (setup.py) ... [?25l[?25hdone
    Requirement already satisfied: six in /usr/local/lib/python3.12/dist-packages (from builtwith) (1.17.0)
    Building wheels for collected packages: builtwith
      Building wheel for builtwith (setup.py) ... [?25l[?25hdone
      Created wheel for builtwith: filename=builtwith-1.3.4-py3-none-any.whl size=36077 sha256=3ba4650ca99bfece957d42fddd61df98b98fbbd7e9e05813538fecbae7b799e9
      Stored in directory: /root/.cache/pip/wheels/7f/2d/b2/606e3df914d4aeeab99c4a4e3e9a61673d2293c2e346db00c8
    Successfully built builtwith
    Installing collected packages: builtwith
    Successfully installed builtwith-1.3.4
    


```python
import builtwith

# Analisis teknologi yang digunakan
res = builtwith.parse('https://www.detik.com/')
print(res)
```

    {'databases': ['Firebase'], 'advertising-networks': ['Google AdSense'], 'tag-managers': ['Google Tag Manager'], 'javascript-frameworks': ['jQuery']}
    

## **Crawling Data**


```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import re

HEADERS = {"User-Agent": "Mozilla/5.0"}

KATEGORI_URLS = {
    "news": "https://news.detik.com/indeks",
    "finance": "https://finance.detik.com/indeks",
    "health": "https://health.detik.com/indeks",
    "sport": "https://sport.detik.com/indeks",
    "hot": "https://hot.detik.com/indeks"
}

def crawl_detik_indeks(max_pages=30, max_berita=200):
    data = {"id": [], "judul": [], "link": [], "kategori": [], "isi": []}
    idx = 0

    for kategori_loop, base_url in KATEGORI_URLS.items():
        print(f"\n=== Ambil kategori: {kategori_loop} ===")

        for page in range(1, max_pages + 1):
            if idx >= max_berita:
                break

            url = f"{base_url}?page={page}"
            try:
                r = requests.get(url, headers=HEADERS, timeout=10)
                r.raise_for_status()
            except Exception as e:
                print(f"Gagal akses {url}: {e}")
                continue

            soup = BeautifulSoup(r.text, "html.parser")
            berita_list = soup.select("h3.media__title a")

            if not berita_list:
                continue

            for berita in berita_list:
                if idx >= max_berita:
                    break

                judul = berita.get_text(strip=True)
                link = berita.get("href")

                if not link or not link.startswith("http"):
                    continue

                # ambil isi berita
                isi = ""
                try:
                    res = requests.get(link, headers=HEADERS, timeout=10)
                    res.raise_for_status()
                    soup_detail = BeautifulSoup(res.text, "html.parser")
                    paragraf = soup_detail.select("div.detail__body-text.itp_bodycontent p")
                    if not paragraf:
                        paragraf = soup_detail.select("div.detail__body-text p")

                    teks_list = []
                    for p in paragraf:
                        teks = p.get_text(" ", strip=True)
                        teks = re.sub(r"\s+", " ", teks.lower())
                        if teks:
                            teks_list.append(teks)

                    isi = " ".join(teks_list)
                except Exception:
                    isi = "(gagal ambil isi berita)"

                idx += 1
                data["id"].append(idx)
                data["judul"].append(judul)
                data["link"].append(link)
                data["kategori"].append(kategori_loop)  # gunakan kategori indeks
                data["isi"].append(isi)

                print(f"[{idx}] {judul} [{kategori_loop}]")

            if idx >= max_berita:
                break

        if idx >= max_berita:
            break

    # simpan ke CSV
    df = pd.DataFrame(data)
    df.to_csv("berita_detik_categorized.csv", index=False, encoding="utf-8-sig")
    print(f"\n[DONE] {len(df)} berita tersimpan di berita_detik_categorized.csv")
    return df


# jalankan: ambil 200 berita campur kategori sesuai indeks
crawl_detik_indeks(max_pages=30, max_berita=200)

```

    
    === Ambil kategori: news ===
    [1] Alfamart Gelar Donor Darah Serentak di 34 Kota, Target 26 Ribu Kantong [news]
    [2] Yusril: Presiden Tak Akan Bentuk Tim Investigasi Independen Usut Demo Ricuh [news]
    [3] Bobby Nasution Wajibkan OPD di Sumut Beri Keterangan Pers Setiap Hari [news]
    [4] Jelang Muktamar X, DPC PPP Se-Jateng Deklarasi Dukung Mardiono [news]
    [5] Ada Diskon Tiket Whoosh untuk Keberangkatan 22-24 September, Cek Infonya! [news]
    [6] Video: Profil Djamari Chaniago, Jenderal Tempur yang Kini Jadi Menko Polkam [news]
    [7] Jadi Kepala Badan Komunikasi Pemerintah, Angga Raka Tetap Wamen Komdigi [news]
    [8] Eropa Percepat Sanksi Energi Rusia di Tengah Tekanan Politik [news]
    [9] Warga Segel Rumah Dapur MBG di Bandung karena Bau dan Beroperasi 24 Jam [news]
    [10] Dunia Hari Ini: Perjanjian Militer Antara Papua Nugini dan Australia Gagal Tercapai [news]
    [11] Video: Massa Demo Ojol di Depan DPR Bubar [news]
    [12] SNBP 2026: Jadwal hingga Ketentuan Umum-Khusus [news]
    [13] Protes Aktivis Ditahan, Admin Gejayan Memanggil Mogok Makan di Rutan Polda [news]
    [14] Rapat Baleg DPR Memanas Saat Dua Pimpinan Debat soal RUU Pemilu [news]
    [15] Ditemukan di Malang, Bima Sedang Jualan Mainan di Klenteng [news]
    [16] SNPMB 2026 Kapan Dibuka? Ini Jadwal Registrasi Akun [news]
    [17] Legislator Gerindra Desak Tinjau Ulang Izin Hutan Tanaman Industri di Babel [news]
    [18] Mensesneg Ungkap Plt Menteri BUMN Bakal Diisi Wamen [news]
    [19] Pertamina Perbarui Website Layanan Informasi Jadi Ramah Disabilitas [news]
    [20] Perwakilan Sudah Temui Legislator, Massa Ojol Mulai Bubarkan Diri dari DPR [news]
    [21] Ekuador Tetapkan Status Darurat Buntut Protes Subsidi BBM Dihapus [news]
    [22] Motor Industrialisasi Rakyat: Jutaan Lapangan Kerja Baru oleh Prabowo [news]
    [23] Seminar PDIP, Megawati Cerita Pernah Masak Nasi Goreng untuk Prabowo [news]
    [24] Tim Reformasi Kepolisian Sedang Disusun, Bakal Diumumkan Pekan Ini [news]
    [25] Kemensos Santuni Korban Unjuk Rasa Tigaraksa, Komitmen Dampingi Keluarga [news]
    [26] Massa Ojol Usai Temui DPR: Katanya Presiden Mau Buat Perpres soal Ojol [news]
    [27] Haris Azhar dan TAUD ke Polda Metro, Minta Kasus Delpedro dkk Disetop [news]
    [28] Video: Momen Prabowo Lantik Erick Thohir Jadi Menpora [news]
    [29] Tak Perlu Lagi Muter Jauh, 63 Jembatan Gantung Siap Dibangun Pemerintah [news]
    [30] Video Dapur MBG di Turangga Bandung Disegel Warga [news]
    [31] Jaksa Tanya Perasaan, Istri Hakim Terdakwa Kasus Suap Ngaku Sudah Hopeless [news]
    [32] Video Kelakar Menteri Pigai di DPR: Saya Gerindra Tapi Tak Punya KTA [news]
    [33] Video: Rekap KPK, Gratifikasi Jadi Kasus Korupsi Paling Tinggi di RI [news]
    [34] Prabowo Bentuk Badan Komunikasi Pemerintah, PCO Bubar? [news]
    [35] Video Komisi XIII soal TNI Jaga Gedung DPR: Kita Kerja Perlu Situasi Aman [news]
    [36] Kapolri Tegaskan Siap Ikuti Kebijakan soal Reformasi Kepolisian [news]
    [37] Viral Pria di Cirebon Pura-pura Tertabrak Lalu Peras Pengendara Mobil [news]
    [38] Daftar Lengkap 3 Kali Reshuffle Kabinet Prabowo-Gibran [news]
    [39] Sidang Vonis Pencucian Uang Nikel Blok Mandiodo Ditunda Sepekan [news]
    [40] Awal Mula Kartu Nama Kacab Bank Jatuh ke Tangan Ken Si Otak Penculikan [news]
    [41] Video: Daftar 11 Pejabat yang Dilantik Prabowo Hari Ini [news]
    [42] Harapan Kapolda Riau Satkamling Tak Cuma Jaga Keamanan Kampung, tapi Kehidupan [news]
    [43] Pansus DPRD DKI: PAD Potensi Rugi Rp 700 M Per Tahun Akibat Parkir Ilegal [news]
    [44] Waka Komisi X DPR Sebut Erick Tak Harus Mundur Ketua PSSI Usai Jadi Menpora [news]
    [45] HUT Ke-15 BNPP, Mendagri Ingatkan Tiga Peran Utama di Perbatasan [news]
    [46] Video Cegah Korupsi Berulang, Wamenkes: Buat Sistem yang Rigit [news]
    [47] Video: Respons Grab soal Tuntutan Demo Ojol Hari Ini [news]
    [48] Didemo Rakyat, Timor Leste Batalkan Mobil Dinas Baru Anggota Parlemen [news]
    [49] Aktivis Gelar Nobar Putusan MK, Kecewa Uji Formil UU TNI Ditolak [news]
    [50] Kronologi 'Bang Jago' Pukuli 2 Pemotor di Bogor Berujung Digeruduk Warga [news]
    [51] Kemensos Gaet KND dan Tokoh Lintas Agama Tingkatkan Layanan Disabilitas [news]
    [52] Prabowo Beri Pangkat Jenderal Kehormatan ke Djamari dan Ahmad Dofiri [news]
    [53] Digeser ke Menpora, Erick Thohir Sebut Bakal Ada Plt Menteri BUMN [news]
    [54] Komisi XIII DPR Usul RUU Hak Cipta Masuk Prolegnas Prioritas 2026 [news]
    [55] Jaksa Gali Aliran Duit Djuyamto: Bangun Kantor Terpadu NU, Nafkahi Istri [news]
    [56] Bima yang Dilaporkan Hilang oleh KontraS Ditemukan di Malang [news]
    [57] MK Batasi Gugatan Buruh Di-PHK Hanya Bisa Diajukan 1 Tahun Usai Mediasi Gagal [news]
    [58] Serang Kota Gaza, Israel Buka Rute Baru untuk Warga Palestina Ngungsi [news]
    [59] Keluarga Ungkap Perubahan Perilaku Kacab Bank Sepekan Sebelum Diculik [news]
    [60] Diguyur Hujan, Massa Ojol Tetap Gelar Aksi di Depan DPR RI [news]
    [61] Dorong Akuntabilitas dan Integritas, Kemnaker Pakai Sistem SMAP & Sikencur [news]
    [62] Trump Kunjungi Inggris, Bahas Kerja Sama Nuklir dan Investasi Teknologi [news]
    [63] Polisi Lacak Pria Pukuli Mobil gegara Parkir di Jl Suryakencana Bogor [news]
    [64] Erick Thohir Digeser ke Menpora, Kursi Menteri BUMN Kosong [news]
    [65] Lengkap! Daftar Nama Menteri Kabinet Prabowo-Gibran Usai Pelantikan Hari Ini [news]
    [66] Pansus DPRD Sidak 2 Titik Parkir Ilegal di Jaktim, Dishub Langsung Segel [news]
    [67] Viva Yoga: 2.000 Tim Ekspedisi Patriot Dikirim ke 154 Daerah Sabang-Merauke [news]
    [68] Komisi III DPR Usul RUU Jabatan Hakim Masuk Prolegnas Prioritas 2026 [news]
    [69] Prabowo Lantik Wamenkop-Wamenaker-Wamenhut, Ini Daftar Namanya [news]
    [70] Alasan Wanita di Depok Bikin Laporan Palsu Dibegal Usai Jual Motor [news]
    [71] Komisi II DPR Usul UU Pemilu hingga MD3 Masuk Prolegnas Prioritas 2026 [news]
    [72] MK Tolak Gugatan YLBHI-KontraS soal UU TNI [news]
    [73] Pansus DPR-Pemerintah Setuju RUU Pengelolaan Ruang Udara Dibawa ke Paripurna [news]
    [74] Sejumlah Ojol Tetap Narik Saat Massa Demo Potongan Aplikator 10 % di DPR [news]
    [75] Ada Apartemen hingga Rumah Tak Laku, KPK Lelang Lagi 10 Desember [news]
    [76] Profil Menko Polkam Djamari Chaniago, Eks Jenderal Tempur Kostrad [news]
    [77] Ahmad Dofiri Jadi Penasihat Khusus Presiden Bidang Kamtibmas-Reformasi Polri [news]
    [78] Lelang KPK, Gelang Emas Naga Koruptor Laku Rp 75 Juta [news]
    [79] Terima Delegasi UEA, Mendagri Bahas Kolaborasi Penguatan SDM [news]
    [80] Sekolah Rakyat Jadi Penuntun Mimpi Khomairoh, Siswa Difabel Asal Malang [news]
    [81] Angga Raka Resmi Jadi Kepala Badan Komunikasi Pemerintah [news]
    [82] Prabowo Resmi Lantik 11 Pejabat: Menko Polkam Djamari Chaniago, Menpora Erick Thohir [news]
    [83] Iran Hukum Gantung Pria yang Dituduh Jadi Mata-mata Mossad [news]
    [84] Cegah Tawuran-Kejahatan Jalanan, Polres Serang Tingkatkan Patroli [news]
    [85] Wanita di Depok Minta Maaf Usai Ngaku Dibegal Padahal Motor Dijual [news]
    [86] Antisipasi Ancaman, Istri dan 2 Anak Kacab Bank Ajukan Perlindungan ke LPSK [news]
    [87] Afriansyah Noor Merapat ke Istana: Doakan Biar Bisa Bantu Presiden [news]
    [88] Harga Beras Mulai Turun, Mentan Amran Sidak Pasar Panorama Bengkulu [news]
    [89] Rumah Terdampak Ledakan Gas Pamulang Tangsel Diperbaiki [news]
    [90] Pemkot Semarang Gerak Cepat Tangani Bencana Alam Dipicu Hujan Lebat [news]
    [91] Pria di Denpasar Bunuh Istri yang Tak Kunjung Sembuh dari Stroke [news]
    [92] Daftar Tokoh Merapat ke Istana Jelang Prabowo Lantik Menteri-Wamen [news]
    [93] Video: Sejumlah Massa Ojol Mulai Datang ke Depan DPR [news]
    [94] Video: Ekuador Membara! Status Darurat Berlaku Gegara Demo BBM [news]
    [95] Dikabarkan Digeser ke PCO, Wamenkomdigi Angga Raka Merapat ke Istana [news]
    [96] Warga Geruduk Rumah 'Bang Jago' yang Pukuli 2 Pemotor di Bogor [news]
    [97] MK Tak Terima Gugatan soal Standar Pendidikan Anggota Polri [news]
    [98] KPK Panggil Dirut Taspen Terkait Kasus Investasi Fiktif [news]
    [99] Erick Thohir Tiba di Istana Jelang Pelantikan Menteri [news]
    [100] Foto Trump-Epstein Mendadak Muncul di Kastil Windsor, 4 Orang Ditangkap [news]
    [101] Foto Trump-Epstein Mendadak Muncul di Kastil Windsor, 4 Orang Ditangkap [news]
    [102] Wanita di Depok Bikin Laporan Palsu Dibegal demi Lunasi Utang Pinjol [news]
    [103] Menteri HAM Tanggapi Laporan KontraS soal 3 Orang Hilang Usai Demo [news]
    [104] Pesan Persatuan dari Paus Leo Saat Kongres Pemuka Agama di Kazakhstan [news]
    [105] Ortu Hakim Meninggal, Sidang Vonis Windu Aji Sutanto Ditunda 24 September [news]
    [106] Ditlantas Polda Metro Latih Ratusan Ojol Pertolongan Pertama Gawat Darurat [news]
    [107] Langkah Densus 88 Cegah Radikalisme Lewat Sosialisasi Kebangsaan [news]
    [108] MK Tak Terima 4 Gugatan UU TNI, Tersisa 1 Perkara Lagi [news]
    [109] Algoritma Kebijakan Prabowo [news]
    [110] Diguyur Hujan, Massa Ojol Suarakan Potongan Aplikator 10% di Depan DPR [news]
    [111] 3 Remaja Putri Asal Bekasi Nyaris Dijual di Malaysia, Diimingi Kerja Salon [news]
    [112] Tokoh-tokoh Ini Tiba di Istana Jelang Pelantikan Pejabat [news]
    [113] Video Primus Yustisio: Malu Anak Orang Kaya-Pejabat Dapat Beasiswa LPDP! [news]
    [114] Polri Gencarkan Gerakan Pangan Murah, Siapkan Sistem Drive-Thru bagi Ojol [news]
    [115] Kepala Legal Wilmar Bantah Urus Suap Rp 60 M untuk Vonis Lepas Kasus Migor [news]
    [116] Wanita di Depok Bikin Laporan Palsu Dibegal, Ternyata Motor Dijual [news]
    [117] Polri Gandeng Komnas HAM-KontraS Terkait Penanganan Orang Hilang saat Demo [news]
    [118] Trump Bilang AS Musnahkan 3 Kapal Narkoba dari Venezuela [news]
    [119] Kapolda Riau: Satkamling Tak Sekadar Pos Jaga, tapi Alarm Keamanan Lingkungan [news]
    [120] Dukung Asta Cita, Krakatau Steel Resmikan Dapur MBG dari Baja Modular [news]
    [121] Video KPK: Indeks Persepsi Korupsi Kita Sangat Rendah, Cuma 37 dari 100 [news]
    [122] Video: Keluarga Minta Pembunuh Kacab Bank Dijerat Pasal Pembunuhan Berencana [news]
    [123] KPK Panggil Eks Bupati Muba Terkait Dugaan Korupsi Infrastruktur [news]
    [124] Video: Ahmad Dofiri Tiba di Istana Jelang Diberi Pangkat Jenderal Kehormatan [news]
    [125] Bukan Lagi Tempat Nunggu Angkot, Halte Tanah Abang Kini Dikuasai Pedagang [news]
    [126] KPK Kembali Periksa Eks Direktur Bina Umrah dan Haji Khusus [news]
    [127] Jaksa Cecar Saksi Kasus Suap Vonis Lepas Migor: Anda Pernah Buang HP? [news]
    [128] Video: El Salvador Sita 1,4 Ton Narkoba yang Mengapung di Laut Pasifik [news]
    [129] Pakai Seragam Dinas, Ahmad Dofiri Tiba di Istana Jelang Pelantikan [news]
    [130] Dua Tahun Berlalu, Korban Gempa Maroko Masih Hidup di Tenda Darurat [news]
    [131] Alvi Pemutilasi Pacar Diamuk dan Diumpat Warga Saat Rekonstruksi di Kosan [news]
    [132] Truk Seruduk 2 Angkot Lagi Ngetem di Bogor, 3 Orang Terluka [news]
    [133] MK Gelar Sidang Putusan 5 Gugatan UU TNI Hari Ini [news]
    [134] Gelar Razia, Pemprov Banten Temukan 86 Kendaraan ASN Nunggak Pajak [news]
    [135] Bareskrim Usul Ada LO Polri di LPSK demi Perkuat Perlindungan Saksi [news]
    [136] Pemobil di Pekanbaru Jadi Tersangka Usai Pukul Pejalan Kaki-Bikin Bayi Jatuh [news]
    [137] Pimpinan Komisi I DPR Ungkap Djamari Chaniago Akan Jadi Menko Polkam [news]
    [138] China Kumpulkan Sekutunya, Bentuk Tatanan Global Saingi AS [news]
    [139] Wamentrans Sebut Pengiriman Transmigran Tergantung Permintaan Pemda [news]
    [140] Ini Sosok Ken Otak Penculikan Kacab Bank demi Bobol Rekening Dormant [news]
    [141] Berkas Sidang Etik 5 Anggota Brimob Pelindas Affan Masih Dilengkapi [news]
    [142] Filipina vs China di Laut China Selatan, 1 Awak Luka Kena Meriam Air [news]
    [143] Polres Meranti Telah Distribusikan 115 Ton Beras Selama Sebulan GPM [news]
    [144] BNPT: Keberagaman Harus Dijaga Sebagai Perekat Bangsa [news]
    [145] RUU Perlindungan Saksi Korban, Kejagung Usul Korban Tak Jadi Alat Bukti [news]
    [146] Rapat Bareng Kajati Sulsel, Legislator Tanya Kasus Uang Palsu UIN Makassar [news]
    [147] Antusiasnya Ibu-ibu Borong Sembako di Gerakan Pangan Murah Polri [news]
    [148] Ibas: Maulid Nabi Inspirasi Peradaban Akhlak dan Persatuan [news]
    [149] Pencuri Kabel Grounding Whoosh Ditangkap Saat Beraksi [news]
    [150] Pekan Tuli Internasional 2025: Tema dan Cara Merayakannya [news]
    [151] MK Tak Terima Gugatan PSU Pilbup Barito Utara, Ini Alasannya [news]
    [152] Waskita Karya Kembali Masuk Top 50 Emiten dalam The 16th IICD CG Award 2025 [news]
    [153] Aksi Damai di Depan Polda Metro Minta Bebaskan Delpedro dkk [news]
    [154] Kapolda Riau Apel Satkamling Garda Terdepan Penjaga Keamanan dan Lingkungan [news]
    [155] Dilelang KPK Lagi, Baju Sutra Goceng Kini Laku Rp 2,6 Juta [news]
    [156] Syarat dan Cara Daftar BPJS Ketenagakerjaan untuk Pekerja Migran [news]
    [157] Video: Pelaku Demo Rusuh di Bandung Dapat Kucuran Dana dari Luar Negeri [news]
    [158] Cerita Warga Pilih Naik MRT Hindari Macet di Fatmawati: Mumpung Rp 1 [news]
    [159] Otak Penculikan Kacab Bank Berkelit soal Sosok Pembisik Rekening Dormant [news]
    [160] Video: 6.118 Personel Gabungan Dikerahkan Kawal Demo Ojol Hari Ini [news]
    [161] Mengapa Komunikasi Pemerintah Lewat Layar Bioskop Penting? [news]
    [162] Siswi Bogor Terjebak di Toilet Bimbel, Damkar Evakuasi [news]
    [163] Cetakan Tangan Emas Presiden Pertama Kazakhstan di Puncak Menara 105 Meter [news]
    [164] Makin Panas! Israel Bombardir Pelabuhan Yaman yang Dikuasai Houthi [news]
    [165] Identitas Kerangka Manusia dalam Pohon Aren Masih Misterius, Tunggu Tes DNA [news]
    [166] MK Tolak Gugatan Hasil PSU Pilgub Papua [news]
    [167] KIP Kuliah 2025: Jadwal, Besaran Bantuan hingga Cara Daftar [news]
    [168] Senangnya Warga Tarif MRT Rp 1 Hari Ini: Sisanya untuk Beli Es Teh [news]
    [169] Penertiban Lahan Reaktivasi KA Rangkasbitung-Pandeglang Dimulai Tahun Depan [news]
    [170] Video: Korban Maafkan Pria Ngaku 'Ring 1 Istana'-Pamer Air Gun di Depok [news]
    [171] Prabowo Lantik Menko Polkam-Menpora di Istana Siang Ini [news]
    [172] Penampakan Predator Seks yang Perkosa 8 Korbannya di Maluku Tenggara [news]
    [173] Kronologi Mahasiswi di Ciracas Dibunuh Pacar ABG gegara Cemburu Buta [news]
    [174] Tenang, Mental Health Dijamin BPJS Kesehatan [news]
    [175] Pramono Jawab Curhat Komeng soal Banjir di Jakarta tapi Jabar Disalahkan [news]
    [176] Bank bjb Ajak Nasabah Nabung Sekaligus Ikut Lari BRODER50 di Lautan Pasir [news]
    [177] Cara Cek Tarif Listrik 2025 Lewat Situs PLN [news]
    [178] Kapolres Pelalawan Cek Pos Kamling, Ajak Warga Aktif Jaga Lingkungan [news]
    [179] Walkot Prabumulih Minta Maaf, Klarifikasi Pencopotan Kepsek-Anak Bawa Mobil [news]
    [180] Trump Mulai Kunjungan Bersejarah di Inggris, Akan Bertemu Raja Charles [news]
    [181] Terungkap Pacar Bunuh Mahasiswi di Kos Ciracas Dipicu Rasa Cemburu [news]
    [182] Kronologi Mobil Boks Ditabrak Avanza hingga Terguling di Tol Jagorawi [news]
    [183] Hadiri The Taste of Papua, Papeda Buat Fatma Saifullah Yusuf Terkesima [news]
    [184] Apa Itu Subjek Data Pribadi? Simak Penjelasannya Menurut UU [news]
    [185] Muncul Isu Kementerian BUMN Akan Dihapus, Legislator Wanti-wanti Hal Ini [news]
    [186] 600 Mobil Melintas di Hari Kedua Uji Coba Jalur Gratis Tol Fatmawati 2 [news]
    [187] Danantara, Strategic Flexibility dan Risk Culture [news]
    [188] AHY Tegaskan Kepastian Hukum & Dorong Pemanfaatan Aset Produktif Lahan [news]
    [189] Predator Seks di Maluku Ancam Sebar Foto Bugil 65 Wanita-Perkosa 8 Korban [news]
    [190] Tinjau Proyek Pengendalian Banjir, AHY Ungkap 3 Faktor Banjir di Bengkulu [news]
    [191] Kapan Fenomena Gerhana Terakhir di Tahun 2025? Catat Tanggalnya [news]
    [192] Ipda Kadek Sumerta, Sosok Peduli 100 Penyandang Disabilitas di Gianyar [news]
    [193] Partner In Crime Juga Ditipu Dukun Pengganda Uang, Kesal Jatah Dikurangi [news]
    [194] Video: Plafon Bandara Sultan Babullah Ternate Ambruk Imbas Hujan Deras [news]
    [195] Alvi Sempat Tertidur di Tangga Kos Usai Mutilasi Pacar Jadi Ratusan Potong [news]
    [196] Patuhi Regulasi WLLP, Perusahaan Bakal Terima Naker Award [news]
    [197] Pendaftaran KJMU 2025 Tahap 2: Jadwal hingga Persyaratan [news]
    [198] Netanyahu Bilang Serangan Israel 'Dibenarkan' karena Qatar Danai Hamas [news]
    [199] Polda Banten Ungkap 577 Kasus Narkoba Sepanjang 2025, 778 Jadi Tersangka [news]
    [200] Video: Heboh Aturan Ijazah Capres-Cawapres Dirahasiakan Berujung Dibatalkan [news]
    
    [DONE] 200 berita tersimpan di berita_detik_categorized.csv
    





  <div id="df-83d55d5b-79c4-427d-99d0-bcc8f0822334" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>judul</th>
      <th>link</th>
      <th>kategori</th>
      <th>isi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Alfamart Gelar Donor Darah Serentak di 34 Kota...</td>
      <td>https://news.detik.com/berita/d-8117096/alfama...</td>
      <td>news</td>
      <td>alfamart kembali menggelar aksi donor darah se...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Yusril: Presiden Tak Akan Bentuk Tim Investiga...</td>
      <td>https://news.detik.com/berita/d-8117082/yusril...</td>
      <td>news</td>
      <td>menteri koordinator bidang hukum, ham, imigras...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Bobby Nasution Wajibkan OPD di Sumut Beri Kete...</td>
      <td>https://news.detik.com/berita/d-8117080/bobby-...</td>
      <td>news</td>
      <td>gubernur sumatera utara (sumut) bobby nasution...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Jelang Muktamar X, DPC PPP Se-Jateng Deklarasi...</td>
      <td>https://news.detik.com/berita/d-8117078/jelang...</td>
      <td>news</td>
      <td>dukungan untuk plt ketua umum partai persatuan...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Ada Diskon Tiket Whoosh untuk Keberangkatan 22...</td>
      <td>https://news.detik.com/berita/d-8117031/ada-di...</td>
      <td>news</td>
      <td>bagi pengguna whoosh rute jakarta-bandung atau...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>195</th>
      <td>196</td>
      <td>Patuhi Regulasi WLLP, Perusahaan Bakal Terima ...</td>
      <td>https://news.detik.com/berita/d-8115918/patuhi...</td>
      <td>news</td>
      <td>kementerian ketenagakerjaan (kemnaker) menging...</td>
    </tr>
    <tr>
      <th>196</th>
      <td>197</td>
      <td>Pendaftaran KJMU 2025 Tahap 2: Jadwal hingga P...</td>
      <td>https://news.detik.com/berita/d-8115915/pendaf...</td>
      <td>news</td>
      <td>pemprov dki jakarta melalui dinas pendidikan j...</td>
    </tr>
    <tr>
      <th>197</th>
      <td>198</td>
      <td>Netanyahu Bilang Serangan Israel 'Dibenarkan' ...</td>
      <td>https://news.detik.com/internasional/d-8115899...</td>
      <td>news</td>
      <td>perdana menteri (pm) israel benjamin netanyahu...</td>
    </tr>
    <tr>
      <th>198</th>
      <td>199</td>
      <td>Polda Banten Ungkap 577 Kasus Narkoba Sepanjan...</td>
      <td>https://news.detik.com/berita/d-8115898/polda-...</td>
      <td>news</td>
      <td>wakapolda banten brigjen hendra wirawan menyam...</td>
    </tr>
    <tr>
      <th>199</th>
      <td>200</td>
      <td>Video: Heboh Aturan Ijazah Capres-Cawapres Dir...</td>
      <td>https://20.detik.com/detikupdate/20250917-2509...</td>
      <td>news</td>
      <td></td>
    </tr>
  </tbody>
</table>
<p>200 rows × 5 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-83d55d5b-79c4-427d-99d0-bcc8f0822334')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-83d55d5b-79c4-427d-99d0-bcc8f0822334 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-83d55d5b-79c4-427d-99d0-bcc8f0822334');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    <div id="df-be48a475-af12-4200-9dd0-a4e5cc8cca9e">
      <button class="colab-df-quickchart" onclick="quickchart('df-be48a475-af12-4200-9dd0-a4e5cc8cca9e')"
                title="Suggest charts"
                style="display:none;">

<svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
     width="24px">
    <g>
        <path d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z"/>
    </g>
</svg>
      </button>

<style>
  .colab-df-quickchart {
      --bg-color: #E8F0FE;
      --fill-color: #1967D2;
      --hover-bg-color: #E2EBFA;
      --hover-fill-color: #174EA6;
      --disabled-fill-color: #AAA;
      --disabled-bg-color: #DDD;
  }

  [theme=dark] .colab-df-quickchart {
      --bg-color: #3B4455;
      --fill-color: #D2E3FC;
      --hover-bg-color: #434B5C;
      --hover-fill-color: #FFFFFF;
      --disabled-bg-color: #3B4455;
      --disabled-fill-color: #666;
  }

  .colab-df-quickchart {
    background-color: var(--bg-color);
    border: none;
    border-radius: 50%;
    cursor: pointer;
    display: none;
    fill: var(--fill-color);
    height: 32px;
    padding: 0;
    width: 32px;
  }

  .colab-df-quickchart:hover {
    background-color: var(--hover-bg-color);
    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);
    fill: var(--button-hover-fill-color);
  }

  .colab-df-quickchart-complete:disabled,
  .colab-df-quickchart-complete:disabled:hover {
    background-color: var(--disabled-bg-color);
    fill: var(--disabled-fill-color);
    box-shadow: none;
  }

  .colab-df-spinner {
    border: 2px solid var(--fill-color);
    border-color: transparent;
    border-bottom-color: var(--fill-color);
    animation:
      spin 1s steps(1) infinite;
  }

  @keyframes spin {
    0% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
      border-left-color: var(--fill-color);
    }
    20% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    30% {
      border-color: transparent;
      border-left-color: var(--fill-color);
      border-top-color: var(--fill-color);
      border-right-color: var(--fill-color);
    }
    40% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-top-color: var(--fill-color);
    }
    60% {
      border-color: transparent;
      border-right-color: var(--fill-color);
    }
    80% {
      border-color: transparent;
      border-right-color: var(--fill-color);
      border-bottom-color: var(--fill-color);
    }
    90% {
      border-color: transparent;
      border-bottom-color: var(--fill-color);
    }
  }
</style>

      <script>
        async function quickchart(key) {
          const quickchartButtonEl =
            document.querySelector('#' + key + ' button');
          quickchartButtonEl.disabled = true;  // To prevent multiple clicks.
          quickchartButtonEl.classList.add('colab-df-spinner');
          try {
            const charts = await google.colab.kernel.invokeFunction(
                'suggestCharts', [key], {});
          } catch (error) {
            console.error('Error during call to suggestCharts:', error);
          }
          quickchartButtonEl.classList.remove('colab-df-spinner');
          quickchartButtonEl.classList.add('colab-df-quickchart-complete');
        }
        (() => {
          let quickchartButtonEl =
            document.querySelector('#df-be48a475-af12-4200-9dd0-a4e5cc8cca9e button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>



