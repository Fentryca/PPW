# Crawlling PTA Universitas Trunojoyo Madura

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import re, sys, time
```


```python
BASE_URL = "https://pta.trunojoyo.ac.id/c_search/byprod"
```

Fungsi


```python
def get_max_page(prodi_id):
    url = f"{BASE_URL}/{prodi_id}/1"
    r = requests.get(url)
    soup = BeautifulSoup(r.content, "html.parser")

    # Cari tombol >> (last page)
    last_page = soup.select_one('ol.pagination a:contains("»")')
    if last_page and "href" in last_page.attrs:
        href = last_page["href"]
        # Pecah URL -> ambil angka terakhir
        max_page = int(href.split("/")[-1])
        return max_page

    # fallback kalau pagination tidak ada
    return 1
```


```python
# Contoh pemakaian
print(get_max_page(10))
```

    172
    

    /usr/local/lib/python3.12/dist-packages/soupsieve/css_parser.py:876: FutureWarning: The pseudo class ':contains' is deprecated, ':-soup-contains' should be used moving forward.
      warnings.warn(  # noqa: B028
    


```python
def print_progress(prodi_id, prodi, current_page, total_pages):
    percent = (current_page / total_pages) * 100
    bar_length = 20
    filled_length = int(bar_length * current_page // total_pages)
    bar = '█' * filled_length + '-' * (bar_length - filled_length)
    sys.stdout.write(f'\r[{prodi_id}] {prodi} - Page {current_page}/{total_pages} [{bar}] {percent:.2f}%')
    sys.stdout.flush()
    if current_page == total_pages:
        sys.stdout.write('\n')
```

## Crawlling semua data PTA


```python
def pta_all():
    start_time = time.time()

    data = {
        "id": [],
        "penulis": [],
        "judul": [],
        "abstrak_id": [],
        "abstrak_en": [],
        "pembimbing_pertama": [],
        "pembimbing_kedua": [],
        "prodi": []
    }

    total_prodi = 1
    total_pages = 0
    max_pages_dict = {}

    # hitung total halaman (untuk tiap prodi)
    for i in range(1, total_prodi + 1):
        max_page = get_max_page(i)
        max_pages_dict[i] = max_page
        total_pages += max_page

    for i in range(1, total_prodi + 1):
        max_page = max_pages_dict[i]
        for j in range(1, max_page + 1):
            url = f"{BASE_URL}/{i}/{j}"
            r = requests.get(url)
            soup = BeautifulSoup(r.content, "html.parser")
            jurnals = soup.select('li[data-cat="#luxury"]')

            isii = soup.select_one('div#begin')
            if not isii:
                continue
            prodi_full = isii.select_one('h2').text.strip()
            prodi = prodi_full.replace("Journal Jurusan ", "")

            for jurnal in jurnals:
                link_keluar = jurnal.select_one('a.gray.button')['href']

                # ambil ID dari link PTA (angka terakhir di URL)
                id_match = re.search(r"/detail/(\d+)", link_keluar)
                pta_id = id_match.group(1) if id_match else None

                response = requests.get(link_keluar)
                soup1 = BeautifulSoup(response.content, "html.parser")
                isi = soup1.select_one('div#content_journal')

                judul = isi.select_one('a.title').text.strip()
                penulis = isi.select_one('span:contains("Penulis")').text.split(' : ')[1]
                pembimbing_pertama = isi.select_one('span:contains("Dosen Pembimbing I")').text.split(' : ')[1]
                pembimbing_kedua = isi.select_one('span:contains("Dosen Pembimbing II")').text.split(' :')[1]

                paragraf = isi.select('p[align="justify"]')
                abstrak_id = paragraf[0].get_text(strip=True) if len(paragraf) > 0 else "N/A"
                abstrak_en = paragraf[1].get_text(strip=True) if len(paragraf) > 1 else "N/A"

                data["id"].append(pta_id)
                data["penulis"].append(penulis)
                data["judul"].append(judul)
                data["abstrak_id"].append(abstrak_id)
                data["abstrak_en"].append(abstrak_en)
                data["pembimbing_pertama"].append(pembimbing_pertama)
                data["pembimbing_kedua"].append(pembimbing_kedua)
                data["prodi"].append(prodi)

            # update progress bar per prodi
            print_progress(i, prodi, j, max_page)

        sys.stdout.write("\n")  # pindah baris setelah 1 prodi selesai

    # simpan ke CSV
    df = pd.DataFrame(data)
    df.to_csv("pta_all.csv", index=False, encoding="utf-8-sig")

    # hitung durasi
    end_time = time.time()
    elapsed = int(end_time - start_time)
    jam, sisa = divmod(elapsed, 3600)
    menit, detik = divmod(sisa, 60)

    # summary
    print("\n✅ Seluruh data berhasil dikumpulkan!")
    print(f"📊 Total entri: {len(df)}")
    print(f"⏱️ Waktu eksekusi: {jam} jam {menit} menit {detik} detik")

    return df
```


```python
pta_all()
```

    /usr/local/lib/python3.12/dist-packages/soupsieve/css_parser.py:876: FutureWarning: The pseudo class ':contains' is deprecated, ':-soup-contains' should be used moving forward.
      warnings.warn(  # noqa: B028
    

    [1] Ilmu Hukum - Page 284/284 [████████████████████] 100.00%
    
    
    ✅ Seluruh data berhasil dikumpulkan!
    📊 Total entri: 1417
    ⏱️ Waktu eksekusi: 2 jam 56 menit 29 detik
    





  <div id="df-b8208b8e-7b33-4e02-b700-dbb901b2c88f" class="colab-df-container">
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
      <th>penulis</th>
      <th>judul</th>
      <th>abstrak_id</th>
      <th>abstrak_en</th>
      <th>pembimbing_pertama</th>
      <th>pembimbing_kedua</th>
      <th>prodi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>080111100012</td>
      <td>Dyah Ayu Citra Seza</td>
      <td>Implementasi Fungsi Legislasi Dewan Perwakilan...</td>
      <td>ABSTRAK\r\n\r\n       Implementasi Fungsi Legi...</td>
      <td>ABSTRACT\r\n       Implementation of Legislati...</td>
      <td>Yudi Widagdo Harimurti, SH., MH</td>
      <td>Safi', SH., MH</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1</th>
      <td>080111100002</td>
      <td>Maulina Nurlaily</td>
      <td>Pertanggungjawaban Pidana Direksi BUMN (Perser...</td>
      <td>Badan Usaha Milik Negara (BUMN) adalah Badan u...</td>
      <td>State Owned Enterprises (SOEs) are business en...</td>
      <td>Tolib Effendi, SH., MH.</td>
      <td>Dr. Eni Suastuti, SH., Mhum.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>2</th>
      <td>070111100060</td>
      <td>Moh. Samsul Hidayat</td>
      <td>Analisis Terhadap Kekosongan Hukum dalam Penga...</td>
      <td>Kasus narkoba tidak henti-hentinya terdengar d...</td>
      <td>Drug cases endlessly heard on television, radi...</td>
      <td>Tolib Effendi, SH., MH.</td>
      <td>Agus Ramdlany, SH., MH.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>3</th>
      <td>090111100077</td>
      <td>TOMMY ADITYA PARLINDUNGAN MARBUN</td>
      <td>PERLINDUNGAN HUKUM BAGI KONSUMEN ATAS PRODUK E...</td>
      <td>Produk elektronik adalah suatu benda bergerak ...</td>
      <td>Electronic products is an object moves through...</td>
      <td>DR. DJULAEKA, S.H., M.HUM</td>
      <td>DR.USWATUN HASANAH, S.H., M. HUM</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>4</th>
      <td>070111200007</td>
      <td>RICA YENA IMADHORA</td>
      <td>TELAAH  KRITIS TENTANG ALASAN HUKUM YANG DIGUN...</td>
      <td></td>
      <td></td>
      <td>Dr. DENI SBY, S. H., M. S.</td>
      <td>SAIFUL ABDULLAH, S. H., M. H.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1412</th>
      <td>150111100130</td>
      <td>DEDY DORES</td>
      <td>PENGKUALIFIKASIAN CHEATER SEBAGAI TINDAK PIDAN...</td>
      <td>Abstrak\n Perbuatan cheater dalam melakukan ch...</td>
      <td>Abstract\n The way of cheater did a cheat in o...</td>
      <td>Aris Hardinanto, S.H., M.H.</td>
      <td></td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1413</th>
      <td>150111100258</td>
      <td>Eko Supriadi</td>
      <td>KUALIFIKASI TINDAK PIDANA ATAS PERBUATAN PELAK...</td>
      <td>Peminjaman dana sistem online dilakukan oleh m...</td>
      <td>The loan funds by online system are carried ou...</td>
      <td>Dr. Erma Rusdiana, S.H.,M.H</td>
      <td></td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1414</th>
      <td>160111100136</td>
      <td>Muslimatul Maghfirah</td>
      <td>KEDUDUKAN HUKUM PEKERJA OUTSOURCING DI DINAS P...</td>
      <td>Abstrak\nTenaga kerja merupakan setiap orang y...</td>
      <td>Abstract\nLabors are those who can work to pro...</td>
      <td>Mishbahul Munir, S.H., M.Hum</td>
      <td></td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1415</th>
      <td>160111100024</td>
      <td>MOH WASIL SYAHRONI</td>
      <td>STAGNANSI HUBUNGAN KELEMBAGAAN DAN KEWENANGAN ...</td>
      <td>Skripsi ini bertujuan untuk menganalisis penti...</td>
      <td>This thesis aims to analyze the stagnation of ...</td>
      <td>Dr. DENI SETYA BAGUS YUHERAWAN, S.H., M.S</td>
      <td></td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1416</th>
      <td>170111100053</td>
      <td>Moch. Steven</td>
      <td>PERUMUSAN SANKSI PIDANA BAGI MASYARAKAT SEKITA...</td>
      <td>ABSTRAK\nAkhir-akhir ini semakin maraknya penc...</td>
      <td>ABSTRACK\nLately, there has been more and more...</td>
      <td>Dr. Wartiningsih, S.H., M.Hum</td>
      <td></td>
      <td>Ilmu Hukum</td>
    </tr>
  </tbody>
</table>
<p>1417 rows × 8 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-b8208b8e-7b33-4e02-b700-dbb901b2c88f')"
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
        document.querySelector('#df-b8208b8e-7b33-4e02-b700-dbb901b2c88f button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-b8208b8e-7b33-4e02-b700-dbb901b2c88f');
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


    <div id="df-2856a504-90db-458d-abe5-170c722dda4c">
      <button class="colab-df-quickchart" onclick="quickchart('df-2856a504-90db-458d-abe5-170c722dda4c')"
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
            document.querySelector('#df-2856a504-90db-458d-abe5-170c722dda4c button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>





```python
def print_progress(prodi_id, prodi, current_page, total_pages):
    percent = (current_page / total_pages) * 100
    bar_length = 20
    filled_length = int(bar_length * current_page // total_pages)
    bar = '█' * filled_length + '-' * (bar_length - filled_length)
    sys.stdout.write(f'\r[{prodi_id}] {prodi} - Page {current_page}/{total_pages} [{bar}] {percent:.2f}%')
    sys.stdout.flush()
    if current_page == total_pages:
        sys.stdout.write('\n\n')

def pta():
    start_time = time.time()  # mulai hitung waktu

    data = {
        "id": [],
        "penulis": [],
        "judul": [],
        "abstrak id": [],
        "abstrak en": [],
        "pembimbing_pertama": [],
        "pembimbing_kedua": [],
        "prodi": [],
    }

    for i in range(1, 42):  # jumlah prodi
        total_pages = 3  # jumlah page
        prodi_name = None

        for j in range(1, total_pages + 1):  # loop page
            url = f"https://pta.trunojoyo.ac.id/c_search/byprod/{i}/{j}"
            r = requests.get(url)
            soup = BeautifulSoup(r.content, "html.parser")
            jurnals = soup.select('li[data-cat="#luxury"]')

            isii = soup.select_one('div#begin')
            if not isii:
                continue
            prodi_full = isii.select_one('h2').text.strip()
            prodi = prodi_full.replace("Journal Jurusan ", "")
            if not prodi_name:
                prodi_name = prodi

            for jurnal in jurnals:
                link = jurnal.select_one('a.gray.button')['href']

                # ambil ID dari link PTA
                id_match = re.search(r"/detail/(\d+)", link)
                pta_id = id_match.group(1) if id_match else None

                response = requests.get(link)
                soup1 = BeautifulSoup(response.content, "html.parser")
                isi = soup1.select_one('div#content_journal')

                # Judul
                judul = isi.select_one('a.title').text

                # Penulis
                penulis = isi.select_one('span:contains("Penulis")').text.split(' : ')[1]

                # Pembimbing Pertama
                pembimbing_pertama = isi.select_one('span:contains("Dosen Pembimbing I")').text.split(' : ')[1]

                # Pembimbing Kedua
                pembimbing_kedua = isi.select_one('span:contains("Dosen Pembimbing II")').text.split(' :')[1]

                # Abstrak
                paragraf = isi.select('p[align="justify"]')
                abstrak = paragraf[0].get_text(strip=True) if len(paragraf) > 0 else "N/A"
                abstract = paragraf[1].get_text(strip=True) if len(paragraf) > 1 else "N/A"

                # simpan data
                data["id"].append(pta_id)
                data["penulis"].append(penulis)
                data["judul"].append(judul)
                data["pembimbing_pertama"].append(pembimbing_pertama)
                data["pembimbing_kedua"].append(pembimbing_kedua)
                data["abstrak id"].append(abstrak)
                data["abstrak en"].append(abstract)
                data["prodi"].append(prodi)

            # update progress bar
            print_progress(i, prodi_name, j, total_pages)

    df = pd.DataFrame(data)
    df.to_csv("pta.csv", index=False, encoding="utf-8-sig")

    end_time = time.time()
    elapsed = int(end_time - start_time)
    jam, sisa = divmod(elapsed, 3600)
    menit, detik = divmod(sisa, 60)

    # summary
    print("\n✅ Seluruh data berhasil dikumpulkan!")
    print(f"📊 Total entri: {len(df)}")
    print(f"⏱️ Waktu eksekusi: {jam} jam {menit} menit {detik} detik")

    return df
```


```python
pta()
```

    [1] Ilmu Hukum - Page 3/3 [████████████████████] 100.00%
    
    [2] Teknologi Industri Pertanian - Page 3/3 [████████████████████] 100.00%
    
    [3] Agribisnis - Page 3/3 [████████████████████] 100.00%
    
    [4] Agroteknologi - Page 3/3 [████████████████████] 100.00%
    
    [5] Ilmu Kelautan - Page 3/3 [████████████████████] 100.00%
    
    [6] Ekonomi Pembangunan - Page 3/3 [████████████████████] 100.00%
    
    [7] Manajemen - Page 3/3 [████████████████████] 100.00%
    
    [8] Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [9] Teknik Industri - Page 3/3 [████████████████████] 100.00%
    
    [10] Teknik Informatika - Page 3/3 [████████████████████] 100.00%
    
    [11] Manajemen Informatika - Page 3/3 [████████████████████] 100.00%
    
    [12] Sosiologi - Page 3/3 [████████████████████] 100.00%
    
    [13] Ilmu Komunikasi - Page 3/3 [████████████████████] 100.00%
    
    [14] Psikologi - Page 3/3 [████████████████████] 100.00%
    
    [15] Sastra Inggris - Page 3/3 [████████████████████] 100.00%
    
    [16] Ekonomi Syariah - Page 3/3 [████████████████████] 100.00%
    
    [17] Hukum Bisnis Syariah - Page 3/3 [████████████████████] 100.00%
    
    [18] Pgsd - Page 3/3 [████████████████████] 100.00%
    
    [19] Teknik Multimedia Dan Jaringan - Page 3/3 [████████████████████] 100.00%
    
    [20] Mekatronika - Page 3/3 [████████████████████] 100.00%
    
    [21] D3 Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [22] Magister Manajemen - Page 3/3 [████████████████████] 100.00%
    
    [23] Teknik Elektro - Page 3/3 [████████████████████] 100.00%
    
    [24] Magister Ilmu Hukum - Page 3/3 [████████████████████] 100.00%
    
    [25] Magister Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [26] D3 Enterpreneurship - Page 3/3 [████████████████████] 100.00%
    
    [27] Pendidikan Bhs Dan Sastra Indonesia - Page 3/3 [████████████████████] 100.00%
    
    [28] Pendidikan Informatika - Page 3/3 [████████████████████] 100.00%
    
    [29] Pendidikan Ipa - Page 3/3 [████████████████████] 100.00%
    
    [30] Pgpaud - Page 3/3 [████████████████████] 100.00%
    
    [31] Sistem Informasi - Page 3/3 [████████████████████] 100.00%
    
    [32] Teknik Mesin - Page 3/3 [████████████████████] 100.00%
    
    [33] Teknik Mekatronika - Page 3/3 [████████████████████] 100.00%
    
    [34] Journal Jurusan - Page 3/3 [████████████████████] 100.00%
    
    [35] Manajemen Sumberdaya Perairan - Page 3/3 [████████████████████] 100.00%
    
    [36] Magister Ilmu Ekonomi - Page 3/3 [████████████████████] 100.00%
    
    [37] Magister Pengelolaan Sumber Daya Alam - Page 3/3 [████████████████████] 100.00%
    
    [38] Pendidikan Profesi Guru - Page 3/3 [████████████████████] 100.00%
    
    [39] Magister Pendidikan Dasar - Page 3/3 [████████████████████] 100.00%
    
    [40] Doktor Pengelolaan Sumber Daya Alam - Page 3/3 [████████████████████] 100.00%
    
    [41] Doktor Ilmu Manajemen - Page 3/3 [████████████████████] 100.00%
    
    
    ✅ Seluruh data berhasil dikumpulkan!
    📊 Total entri: 481
    ⏱️ Waktu eksekusi: 0 jam 50 menit 23 detik
    





  <div id="df-fcd42ca4-e724-42bc-a692-d1e58d9c8d0d" class="colab-df-container">
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
      <th>penulis</th>
      <th>judul</th>
      <th>abstrak id</th>
      <th>abstrak en</th>
      <th>pembimbing_pertama</th>
      <th>pembimbing_kedua</th>
      <th>prodi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>080111100012</td>
      <td>Dyah Ayu Citra Seza</td>
      <td>Implementasi Fungsi Legislasi Dewan Perwakilan...</td>
      <td>ABSTRAK\r\n\r\n       Implementasi Fungsi Legi...</td>
      <td>ABSTRACT\r\n       Implementation of Legislati...</td>
      <td>Yudi Widagdo Harimurti, SH., MH</td>
      <td>Safi', SH., MH</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>1</th>
      <td>080111100002</td>
      <td>Maulina Nurlaily</td>
      <td>Pertanggungjawaban Pidana Direksi BUMN (Perser...</td>
      <td>Badan Usaha Milik Negara (BUMN) adalah Badan u...</td>
      <td>State Owned Enterprises (SOEs) are business en...</td>
      <td>Tolib Effendi, SH., MH.</td>
      <td>Dr. Eni Suastuti, SH., Mhum.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>2</th>
      <td>070111100060</td>
      <td>Moh. Samsul Hidayat</td>
      <td>Analisis Terhadap Kekosongan Hukum dalam Penga...</td>
      <td>Kasus narkoba tidak henti-hentinya terdengar d...</td>
      <td>Drug cases endlessly heard on television, radi...</td>
      <td>Tolib Effendi, SH., MH.</td>
      <td>Agus Ramdlany, SH., MH.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>3</th>
      <td>090111100077</td>
      <td>TOMMY ADITYA PARLINDUNGAN MARBUN</td>
      <td>PERLINDUNGAN HUKUM BAGI KONSUMEN ATAS PRODUK E...</td>
      <td>Produk elektronik adalah suatu benda bergerak ...</td>
      <td>Electronic products is an object moves through...</td>
      <td>DR. DJULAEKA, S.H., M.HUM</td>
      <td>DR.USWATUN HASANAH, S.H., M. HUM</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>4</th>
      <td>070111200007</td>
      <td>RICA YENA IMADHORA</td>
      <td>TELAAH  KRITIS TENTANG ALASAN HUKUM YANG DIGUN...</td>
      <td></td>
      <td></td>
      <td>Dr. DENI SBY, S. H., M. S.</td>
      <td>SAIFUL ABDULLAH, S. H., M. H.</td>
      <td>Ilmu Hukum</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>476</th>
      <td>160281100013</td>
      <td>Lisa Sri rahmatullah, S. Sos. I</td>
      <td>Dampak Sosial Ekonomi Pariwisata Religi Makam ...</td>
      <td>Penelitian ini bertujuan untuk mengetahui baga...</td>
      <td>The purpose of this study is to analyze the so...</td>
      <td>Dr. Diah Wahyuningsih, S.E., M.Si.</td>
      <td>Dr. Eni Sri Rahayuningsih, S.E., M.E.</td>
      <td>Magister Ilmu Ekonomi</td>
    </tr>
    <tr>
      <th>477</th>
      <td>160281100002</td>
      <td>Indah Ainun Nikmah</td>
      <td>Peranan Zakat Produktif Dalam Meningkatkan Eko...</td>
      <td>Peranan Zakat Produktif dalam Meningkatkan Eko...</td>
      <td>The Role of Productive Zakat in Improving Must...</td>
      <td>Dr. Kurniyati Indahsari, M.Si</td>
      <td>Dr. Abdur Rahman, S.Ag. MEI</td>
      <td>Magister Ilmu Ekonomi</td>
    </tr>
    <tr>
      <th>478</th>
      <td>170361100010</td>
      <td>ahmad syaiful umam</td>
      <td>KARAKTERISASI DAN KOLEKSI PLASMA NUTFAH UNTUK ...</td>
      <td>Madura merupakan salah satu wilayah pemasok ko...</td>
      <td>Madura is one of the regions supplying horticu...</td>
      <td>Dr. Ir. Gita Pawana, M.Si</td>
      <td>Dr. Ir. Hj. SIti Fatimah, M.Si</td>
      <td>Magister Pengelolaan Sumber Daya Alam</td>
    </tr>
    <tr>
      <th>479</th>
      <td>170361100001</td>
      <td>Siti Holifah</td>
      <td>PENGOLAHAN LIMBAH AIR REBUSAN IKAN TERI MENJAD...</td>
      <td>Ikan Teri perlu penanganan serius pasca panen ...</td>
      <td>Anchovy needs serious handling after harvest b...</td>
      <td>Dr.Apri Arisandi,S.Pi.,M.Si.</td>
      <td>Dr.Ir.H.Asfan,MP.</td>
      <td>Magister Pengelolaan Sumber Daya Alam</td>
    </tr>
    <tr>
      <th>480</th>
      <td>170361100003</td>
      <td>Mohammad Maskur</td>
      <td>STRATEGI PENGEMBANGAN MAKANAN DAN MINUMAN KHAS...</td>
      <td>Makanan dan minuman khas merupakan ciri dari k...</td>
      <td>Typical food and drinks are characteristic of ...</td>
      <td>Dr. Akhmad Farid, S.Pi., MT</td>
      <td>Dr. Apri Arisandi, S.Pi., M.Si</td>
      <td>Magister Pengelolaan Sumber Daya Alam</td>
    </tr>
  </tbody>
</table>
<p>481 rows × 8 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-fcd42ca4-e724-42bc-a692-d1e58d9c8d0d')"
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
        document.querySelector('#df-fcd42ca4-e724-42bc-a692-d1e58d9c8d0d button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-fcd42ca4-e724-42bc-a692-d1e58d9c8d0d');
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


    <div id="df-d2c27b62-551f-4017-9000-ac4f1ef9c328">
      <button class="colab-df-quickchart" onclick="quickchart('df-d2c27b62-551f-4017-9000-ac4f1ef9c328')"
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
            document.querySelector('#df-d2c27b62-551f-4017-9000-ac4f1ef9c328 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>




## Page dan Link Keluar


```python
def print_progress(prodi_id, prodi, current_page, total_pages):
    percent = (current_page / total_pages) * 100
    bar_length = 20
    filled_length = int(bar_length * current_page // total_pages)
    bar = '█' * filled_length + '-' * (bar_length - filled_length)
    sys.stdout.write(f'\r[{prodi_id}] {prodi} - Page {current_page}/{total_pages} [{bar}] {percent:.2f}%')
    sys.stdout.flush()
    if current_page == total_pages:
        sys.stdout.write('\n\n')

def pta_links():
    start_time = time.time()  # mulai hitung waktu

    data = {
        "no": [],
        "page": [],
        "link_keluar": []
    }

    no = 1  # nomor urut

    for i in range(1, 42):  # jumlah prodi
        total_pages = 3  # jumlah page
        prodi_name = None

        for j in range(1, total_pages + 1):  # loop page
            url = f"https://pta.trunojoyo.ac.id/c_search/byprod/{i}/{j}"
            r = requests.get(url)
            soup = BeautifulSoup(r.content, "html.parser")
            jurnals = soup.select('li[data-cat="#luxury"]')

            isii = soup.select_one('div#begin')
            if not isii:
                continue
            prodi_full = isii.select_one('h2').text.strip()
            prodi = prodi_full.replace("Journal Jurusan ", "")
            if not prodi_name:
                prodi_name = prodi

            for jurnal in jurnals:
                link = jurnal.select_one('a.gray.button')['href']

                data["no"].append(no)
                data["page"].append(url)          # link page
                data["link_keluar"].append(link)  # link detail
                no += 1

            # update progress bar
            print_progress(i, prodi_name, j, total_pages)

    df = pd.DataFrame(data)
    df.to_csv("pta_links.csv", index=False)

    end_time = time.time()
    elapsed = int(end_time - start_time)
    jam, sisa = divmod(elapsed, 3600)
    menit, detik = divmod(sisa, 60)

    # summary
    print("\n✅ Seluruh link berhasil dikumpulkan!")
    print(f"📊 Total entri: {len(df)}")
    print(f"⏱️ Waktu eksekusi: {jam} jam {menit} menit {detik} detik")

    return df
```


```python
pta_links()
```

    [1] Ilmu Hukum - Page 3/3 [████████████████████] 100.00%
    
    [2] Teknologi Industri Pertanian - Page 3/3 [████████████████████] 100.00%
    
    [3] Agribisnis - Page 3/3 [████████████████████] 100.00%
    
    [4] Agroteknologi - Page 3/3 [████████████████████] 100.00%
    
    [5] Ilmu Kelautan - Page 3/3 [████████████████████] 100.00%
    
    [6] Ekonomi Pembangunan - Page 3/3 [████████████████████] 100.00%
    
    [7] Manajemen - Page 3/3 [████████████████████] 100.00%
    
    [8] Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [9] Teknik Industri - Page 3/3 [████████████████████] 100.00%
    
    [10] Teknik Informatika - Page 3/3 [████████████████████] 100.00%
    
    [11] Manajemen Informatika - Page 3/3 [████████████████████] 100.00%
    
    [12] Sosiologi - Page 3/3 [████████████████████] 100.00%
    
    [13] Ilmu Komunikasi - Page 3/3 [████████████████████] 100.00%
    
    [14] Psikologi - Page 3/3 [████████████████████] 100.00%
    
    [15] Sastra Inggris - Page 3/3 [████████████████████] 100.00%
    
    [16] Ekonomi Syariah - Page 3/3 [████████████████████] 100.00%
    
    [17] Hukum Bisnis Syariah - Page 3/3 [████████████████████] 100.00%
    
    [18] Pgsd - Page 3/3 [████████████████████] 100.00%
    
    [19] Teknik Multimedia Dan Jaringan - Page 3/3 [████████████████████] 100.00%
    
    [20] Mekatronika - Page 3/3 [████████████████████] 100.00%
    
    [21] D3 Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [22] Magister Manajemen - Page 3/3 [████████████████████] 100.00%
    
    [23] Teknik Elektro - Page 3/3 [████████████████████] 100.00%
    
    [24] Magister Ilmu Hukum - Page 3/3 [████████████████████] 100.00%
    
    [25] Magister Akuntansi - Page 3/3 [████████████████████] 100.00%
    
    [26] D3 Enterpreneurship - Page 3/3 [████████████████████] 100.00%
    
    [27] Pendidikan Bhs Dan Sastra Indonesia - Page 3/3 [████████████████████] 100.00%
    
    [28] Pendidikan Informatika - Page 3/3 [████████████████████] 100.00%
    
    [29] Pendidikan Ipa - Page 3/3 [████████████████████] 100.00%
    
    [30] Pgpaud - Page 3/3 [████████████████████] 100.00%
    
    [31] Sistem Informasi - Page 3/3 [████████████████████] 100.00%
    
    [32] Teknik Mesin - Page 3/3 [████████████████████] 100.00%
    
    [33] Teknik Mekatronika - Page 3/3 [████████████████████] 100.00%
    
    [34] Journal Jurusan - Page 3/3 [████████████████████] 100.00%
    
    [35] Manajemen Sumberdaya Perairan - Page 3/3 [████████████████████] 100.00%
    
    [36] Magister Ilmu Ekonomi - Page 3/3 [████████████████████] 100.00%
    
    [37] Magister Pengelolaan Sumber Daya Alam - Page 3/3 [████████████████████] 100.00%
    
    [38] Pendidikan Profesi Guru - Page 3/3 [████████████████████] 100.00%
    
    [39] Magister Pendidikan Dasar - Page 3/3 [████████████████████] 100.00%
    
    [40] Doktor Pengelolaan Sumber Daya Alam - Page 3/3 [████████████████████] 100.00%
    
    [41] Doktor Ilmu Manajemen - Page 3/3 [████████████████████] 100.00%
    
    
    ✅ Seluruh link berhasil dikumpulkan!
    📊 Total entri: 481
    ⏱️ Waktu eksekusi: 0 jam 10 menit 32 detik
    





  <div id="df-84b9f737-427d-4697-b8f1-438d1314d759" class="colab-df-container">
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
      <th>no</th>
      <th>page</th>
      <th>link_keluar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/1/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/080...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/1/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/080...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/1/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/070...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/1/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/090...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/1/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/070...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>476</th>
      <td>477</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/36/2</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/160...</td>
    </tr>
    <tr>
      <th>477</th>
      <td>478</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/36/2</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/160...</td>
    </tr>
    <tr>
      <th>478</th>
      <td>479</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/37/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/170...</td>
    </tr>
    <tr>
      <th>479</th>
      <td>480</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/37/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/170...</td>
    </tr>
    <tr>
      <th>480</th>
      <td>481</td>
      <td>https://pta.trunojoyo.ac.id/c_search/byprod/37/1</td>
      <td>https://pta.trunojoyo.ac.id/welcome/detail/170...</td>
    </tr>
  </tbody>
</table>
<p>481 rows × 3 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-84b9f737-427d-4697-b8f1-438d1314d759')"
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
        document.querySelector('#df-84b9f737-427d-4697-b8f1-438d1314d759 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-84b9f737-427d-4697-b8f1-438d1314d759');
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


    <div id="df-f7ddf469-6e13-4062-b836-2ec8084999e8">
      <button class="colab-df-quickchart" onclick="quickchart('df-f7ddf469-6e13-4062-b836-2ec8084999e8')"
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
            document.querySelector('#df-f7ddf469-6e13-4062-b836-2ec8084999e8 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>



