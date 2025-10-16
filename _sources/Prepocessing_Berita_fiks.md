## Prepocessing_Berita
#**Labeling Manual data Berita**


```python
from google.colab import drive
import pandas as pd

# 1️⃣ Sambungkan Google Drive
drive.mount('/content/drive')

# 2️⃣ Path ke file CSV kamu
PATH = '/content/drive/MyDrive/Semester 7/PPW/berita_detik.csv'

# 3️⃣ Baca file
df = pd.read_csv(PATH)

# 4️⃣ Pastikan kolom kategori ada
if 'kategori' not in df.columns:
    df['kategori'] = 'Belum dikategorikan'

# 5️⃣ Definisi kata kunci per kategori
keywords = {
    'Finance': ['ihsg', 'rupiah', 'saham', 'investasi', 'bank', 'bursa', 'inflasi', 'ekonomi'],
    'Politik': ['presiden', 'menteri', 'politik', 'dpr', 'pemilu', 'partai', 'prabowo', 'jokowi'],
    'Olahraga': ['liga', 'piala', 'pertandingan', 'timnas', 'gol', 'fifa', 'sepak bola', 'persib', 'arema'],
    'Kesehatan': ['vaksin', 'covid', 'rumah sakit', 'virus', 'dokter', 'kesehatan', 'obat', 'bpjs'],
    'Makanan': ['resep', 'kuliner', 'restoran', 'makanan', 'minuman', 'menu', 'warung', 'kafe'],
}

# 6️⃣ Fungsi deteksi kategori
def detect_category(text):
    if not isinstance(text, str):
        return 'News'
    text_lower = text.lower()
    for kategori, kata_list in keywords.items():
        if any(kata in text_lower for kata in kata_list):
            return kategori
    return 'News'  # fallback ke News

# 7️⃣ Terapkan ke kolom judul + isi
df['kategori'] = df.apply(
    lambda row: detect_category(str(row['judul']) + ' ' + str(row['isi'])), axis=1
)

# 8️⃣ Simpan hasilnya ke file baru
output_path = '/content/drive/MyDrive/Semester 7/PPW/berita_detik_label_otomatis.csv'
df.to_csv(output_path, index=False)

print("✅ Labeling otomatis selesai! Semua yang tidak terdeteksi akan masuk kategori 'News'")
print("📁 Hasil disimpan di:", output_path)

# 9️⃣ Contoh hasil
print("\n🔍 Contoh hasil:")
print(df[['judul', 'kategori']].head(10))

```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    ✅ Labeling otomatis selesai! Semua yang tidak terdeteksi akan masuk kategori 'News'
    📁 Hasil disimpan di: /content/drive/MyDrive/Semester 7/PPW/berita_detik_label_otomatis.csv
    
    🔍 Contoh hasil:
                                                   judul   kategori
    0  Alfamart Gelar Donor Darah Serentak di 34 Kota...  Kesehatan
    1  Yusril: Presiden Tak Akan Bentuk Tim Investiga...    Politik
    2  Bobby Nasution Wajibkan OPD di Sumut Beri Kete...  Kesehatan
    3  Jelang Muktamar X, DPC PPP Se-Jateng Deklarasi...    Politik
    4  Ada Diskon Tiket Whoosh untuk Keberangkatan 22...       News
    5  Video: Profil Djamari Chaniago, Jenderal Tempu...       News
    6  Jadi Kepala Badan Komunikasi Pemerintah, Angga...    Politik
    7  Eropa Percepat Sanksi Energi Rusia di Tengah T...    Finance
    8  Warga Segel Rumah Dapur MBG di Bandung karena ...       News
    9  Dunia Hari Ini: Perjanjian Militer Antara Papu...    Politik
    

## **Prepocessing**


```python
# 1. Import library
import pandas as pd
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC
from sklearn.metrics import classification_report, accuracy_score
from imblearn.over_sampling import RandomOverSampler
from collections import Counter
import joblib
import re

```


```python
pip install Sastrawi

```

    Collecting Sastrawi
      Downloading Sastrawi-1.0.1-py2.py3-none-any.whl.metadata (909 bytes)
    Downloading Sastrawi-1.0.1-py2.py3-none-any.whl (209 kB)
    [?25l   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m0.0/209.7 kB[0m [31m?[0m eta [36m-:--:--[0m
[2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m209.7/209.7 kB[0m [31m11.3 MB/s[0m eta [36m0:00:00[0m
    [?25hInstalling collected packages: Sastrawi
    Successfully installed Sastrawi-1.0.1
    

**Input Data**


```python
import pandas as pd
from google.colab import drive

# Mount Google Drive
drive.mount('/content/drive')

# Path ke file Excel (ganti sesuai lokasi file di Drive-mu)
file_path = "/content/drive/MyDrive/Semester 7/PPW/berita_detik_label_otomatis.csv"

# Baca file Excel
df = pd.read_csv(file_path)

# Tampilkan nama kolom
print("Nama-nama kolom:")
print(df.columns.tolist())

# Tampilkan 100 baris pertama
print("\nContoh 100 baris pertama:")
print(df.head(100))

```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    Nama-nama kolom:
    ['id', 'judul', 'link', 'kategori', 'isi']
    
    Contoh 100 baris pertama:
         id                                              judul  \
    0     1  Alfamart Gelar Donor Darah Serentak di 34 Kota...   
    1     2  Yusril: Presiden Tak Akan Bentuk Tim Investiga...   
    2     3  Bobby Nasution Wajibkan OPD di Sumut Beri Kete...   
    3     4  Jelang Muktamar X, DPC PPP Se-Jateng Deklarasi...   
    4     5  Ada Diskon Tiket Whoosh untuk Keberangkatan 22...   
    ..  ...                                                ...   
    95   96  Warga Geruduk Rumah 'Bang Jago' yang Pukuli 2 ...   
    96   97  MK Tak Terima Gugatan soal Standar Pendidikan ...   
    97   98  KPK Panggil Dirut Taspen Terkait Kasus Investa...   
    98   99  Erick Thohir Tiba di Istana Jelang Pelantikan ...   
    99  100  Foto Trump-Epstein Mendadak Muncul di Kastil W...   
    
                                                     link   kategori  \
    0   https://news.detik.com/berita/d-8117096/alfama...  Kesehatan   
    1   https://news.detik.com/berita/d-8117082/yusril...    Politik   
    2   https://news.detik.com/berita/d-8117080/bobby-...  Kesehatan   
    3   https://news.detik.com/berita/d-8117078/jelang...    Politik   
    4   https://news.detik.com/berita/d-8117031/ada-di...       News   
    ..                                                ...        ...   
    95  https://news.detik.com/berita/d-8116498/warga-...   Olahraga   
    96  https://news.detik.com/berita/d-8116497/mk-tak...    Politik   
    97  https://news.detik.com/berita/d-8116484/kpk-pa...    Finance   
    98  https://news.detik.com/berita/d-8116478/erick-...    Politik   
    99  https://news.detik.com/internasional/d-8116477...    Politik   
    
                                                      isi  
    0   alfamart kembali menggelar aksi donor darah se...  
    1   menteri koordinator bidang hukum, ham, imigras...  
    2   gubernur sumatera utara (sumut) bobby nasution...  
    3   dukungan untuk plt ketua umum partai persatuan...  
    4   bagi pengguna whoosh rute jakarta-bandung atau...  
    ..                                                ...  
    95  rumah seorang pria berinisial ps (27) di kelur...  
    96  mahkamah konstitusi (mk) tidak menerima gugata...  
    97  kpk memanggil direktur utama (dirut) pt taspen...  
    98  tokoh-tokoh yang merapat ke istana negara menj...  
    99  kepolisian inggris menangkap empat orang setel...  
    
    [100 rows x 5 columns]
    

**Prepocessing**


```python
# Stopword lokal
stopword_indonesia = set([
    "yang", "dan", "di", "ke", "dari", "ini", "itu", "untuk", "dengan", "karena",
    "ada", "saya", "kami", "kita", "mereka", "pada", "adalah", "juga", "tidak",
    "ya", "kok", "loh", "banget", "sih", "jadi", "udah", "lagi", "aja", "dong", "nih"
])

# Normalisasi kata
normalisasi_kata = {
    "bangeettt": "banget", "bgt": "banget", "bgtt": "banget",
    "gk": "tidak", "ga": "tidak", "nggak": "tidak",
    "dr": "dari", "tp": "tapi", "tdk": "tidak",
    "sy": "saya", "lg": "lagi"
}

def normalize_kata(tokens):
    return [normalisasi_kata.get(word, word) for word in tokens]

def simple_tokenize(text):
    return text.split()

def preprocess(text):
    text = text.lower()
    text = re.sub(r'[^a-z\s]', '', text)
    tokens = simple_tokenize(text)
    tokens = normalize_kata(tokens)
    tokens = [word for word in tokens if word not in stopword_indonesia and len(word) > 2]
    return " ".join(tokens)

# Terapkan preprocessing
df['preprocessed'] = df['isi'].astype(str).apply(preprocess)
```


```python
import re
import pandas as pd
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory

# Buat stemmer Sastrawi
factory = StemmerFactory()
stemmer = factory.create_stemmer()

# Ambil 5 data awal
sample_data = df['isi'].astype(str).head(5)

# List untuk simpan hasil
stopwords_removed = []
cleaning = []
normalized = []
stemming = []
tokenizing = []

for text in sample_data:
    # Tokenisasi awal (pisah kata)
    tokens = text.split()

    # Stopword removal
    tokens_sw = [word for word in tokens if word not in stopword_indonesia and len(word) > 2]
    stopwords_removed.append(tokens_sw)

    # Cleaning (hapus karakter non-huruf)
    tokens_clean = [re.sub(r'[^a-zA-Z]', '', word) for word in tokens_sw if re.sub(r'[^a-zA-Z]', '', word) != ""]
    cleaning.append(tokens_clean)

    # Normalisasi (pembakuan kata)
    tokens_norm = [normalisasi_kata.get(word.lower(), word.lower()) for word in tokens_clean]
    normalized.append(tokens_norm)

    # Stemming
    tokens_stem = [stemmer.stem(word) for word in tokens_norm]
    stemming.append(tokens_stem)

    # Tokenisasi akhir
    tokenizing.append(tokens_stem)

# Simpan hasil ke DataFrame
tahapan_df = pd.DataFrame({
    'Asli': sample_data,
    'Stopword Removal': stopwords_removed,
    'Cleaning': cleaning,
    'Normalisasi (Ejaan Baku)': normalized,
    'Stemming': stemming,
    'Tokenisasi Akhir': tokenizing
})

# Tampilkan hasil
from IPython.display import display
display(tahapan_df)

```



  <div id="df-fc954713-1122-4320-b9bf-eb01b4c7cb8c" class="colab-df-container">
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
      <th>Asli</th>
      <th>Stopword Removal</th>
      <th>Cleaning</th>
      <th>Normalisasi (Ejaan Baku)</th>
      <th>Stemming</th>
      <th>Tokenisasi Akhir</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alfamart kembali menggelar aksi donor darah se...</td>
      <td>[alfamart, kembali, menggelar, aksi, donor, da...</td>
      <td>[alfamart, kembali, menggelar, aksi, donor, da...</td>
      <td>[alfamart, kembali, menggelar, aksi, donor, da...</td>
      <td>[alfamart, kembali, gelar, aksi, donor, darah,...</td>
      <td>[alfamart, kembali, gelar, aksi, donor, darah,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>menteri koordinator bidang hukum, ham, imigras...</td>
      <td>[menteri, koordinator, bidang, hukum,, ham,, i...</td>
      <td>[menteri, koordinator, bidang, hukum, ham, imi...</td>
      <td>[menteri, koordinator, bidang, hukum, ham, imi...</td>
      <td>[menteri, koordinator, bidang, hukum, ham, imi...</td>
      <td>[menteri, koordinator, bidang, hukum, ham, imi...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>gubernur sumatera utara (sumut) bobby nasution...</td>
      <td>[gubernur, sumatera, utara, (sumut), bobby, na...</td>
      <td>[gubernur, sumatera, utara, sumut, bobby, nasu...</td>
      <td>[gubernur, sumatera, utara, sumut, bobby, nasu...</td>
      <td>[gubernur, sumatera, utara, sumut, bobby, nasu...</td>
      <td>[gubernur, sumatera, utara, sumut, bobby, nasu...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dukungan untuk plt ketua umum partai persatuan...</td>
      <td>[dukungan, plt, ketua, umum, partai, persatuan...</td>
      <td>[dukungan, plt, ketua, umum, partai, persatuan...</td>
      <td>[dukungan, plt, ketua, umum, partai, persatuan...</td>
      <td>[dukung, plt, ketua, umum, partai, satu, bangu...</td>
      <td>[dukung, plt, ketua, umum, partai, satu, bangu...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bagi pengguna whoosh rute jakarta-bandung atau...</td>
      <td>[bagi, pengguna, whoosh, rute, jakarta-bandung...</td>
      <td>[bagi, pengguna, whoosh, rute, jakartabandung,...</td>
      <td>[bagi, pengguna, whoosh, rute, jakartabandung,...</td>
      <td>[bagi, guna, whoosh, rute, jakartabandung, ata...</td>
      <td>[bagi, guna, whoosh, rute, jakartabandung, ata...</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-fc954713-1122-4320-b9bf-eb01b4c7cb8c')"
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
        document.querySelector('#df-fc954713-1122-4320-b9bf-eb01b4c7cb8c button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-fc954713-1122-4320-b9bf-eb01b4c7cb8c');
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


    <div id="df-467edd87-4a5e-4af9-8c85-bfdee95a5b43">
      <button class="colab-df-quickchart" onclick="quickchart('df-467edd87-4a5e-4af9-8c85-bfdee95a5b43')"
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
            document.querySelector('#df-467edd87-4a5e-4af9-8c85-bfdee95a5b43 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

  <div id="id_95ba6c89-01aa-4eef-af64-da7bc648fe81">
    <style>
      .colab-df-generate {
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

      .colab-df-generate:hover {
        background-color: #E2EBFA;
        box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
        fill: #174EA6;
      }

      [theme=dark] .colab-df-generate {
        background-color: #3B4455;
        fill: #D2E3FC;
      }

      [theme=dark] .colab-df-generate:hover {
        background-color: #434B5C;
        box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
        filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
        fill: #FFFFFF;
      }
    </style>
    <button class="colab-df-generate" onclick="generateWithVariable('tahapan_df')"
            title="Generate code using this dataframe."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px"viewBox="0 0 24 24"
       width="24px">
    <path d="M7,19H8.4L18.45,9,17,7.55,7,17.6ZM5,21V16.75L18.45,3.32a2,2,0,0,1,2.83,0l1.4,1.43a1.91,1.91,0,0,1,.58,1.4,1.91,1.91,0,0,1-.58,1.4L9.25,21ZM18.45,9,17,7.55Zm-12,3A5.31,5.31,0,0,0,4.9,8.1,5.31,5.31,0,0,0,1,6.5,5.31,5.31,0,0,0,4.9,4.9,5.31,5.31,0,0,0,6.5,1,5.31,5.31,0,0,0,8.1,4.9,5.31,5.31,0,0,0,12,6.5,5.46,5.46,0,0,0,6.5,12Z"/>
  </svg>
    </button>
    <script>
      (() => {
      const buttonEl =
        document.querySelector('#id_95ba6c89-01aa-4eef-af64-da7bc648fe81 button.colab-df-generate');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      buttonEl.onclick = () => {
        google.colab.notebook.generateWithVariable('tahapan_df');
      }
      })();
    </script>
  </div>

    </div>
  </div>




```python
import re
import pandas as pd
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory
from google.colab import drive

# Mount Google Drive
drive.mount('/content/drive')

# Buat stemmer
factory = StemmerFactory()
stemmer = factory.create_stemmer()

# Ambil semua data (ubah ke string dulu)
sample_data = df['isi'].astype(str)

# List untuk menyimpan hasil
stopwords_removed = []
cleaning = []
normalized = []
stemming = []
tokenizing = []

for text in sample_data:
    # Tokenisasi awal
    tokens = text.split()

    # Stopword removal
    tokens_sw = [word for word in tokens if word.lower() not in stopword_indonesia and len(word) > 2]
    stopwords_removed.append(tokens_sw)

    # Cleaning (hapus karakter non-huruf)
    tokens_clean = [re.sub(r'[^a-zA-Z]', '', word) for word in tokens_sw if re.sub(r'[^a-zA-Z]', '', word) != ""]
    cleaning.append(tokens_clean)

    # Normalisasi (pembakuan kata)
    tokens_norm = [normalisasi_kata.get(word.lower(), word.lower()) for word in tokens_clean]
    normalized.append(tokens_norm)

    # Stemming
    tokens_stem = [stemmer.stem(word) for word in tokens_norm]
    stemming.append(tokens_stem)

    # Tokenisasi akhir
    tokenizing.append(tokens_stem)

# Gabungkan ke DataFrame
tahapan_df = pd.DataFrame({
    'Teks Asli': sample_data,
    'Setelah Stopword Removal': stopwords_removed,
    'Setelah Cleaning': cleaning,
    'Setelah Normalisasi': normalized,
    'Setelah Stemming': stemming,
    'Tokenisasi Akhir': tokenizing
})

# Tentukan lokasi penyimpanan di Google Drive
save_path = "/content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing_berita.csv"

# Simpan otomatis ke Drive
tahapan_df.to_csv(save_path, index=False, encoding='utf-8-sig')

print(f"✅ File berhasil disimpan di {save_path}")

```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    ✅ File berhasil disimpan di /content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing_berita.csv
    
