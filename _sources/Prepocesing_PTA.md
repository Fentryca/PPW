## Prepocessing PTA

Import Library


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

Input Data


```python
import pandas as pd
from google.colab import drive

# Mount Google Drive
drive.mount('/content/drive')

# Path ke file Excel (ganti sesuai lokasi file di Drive-mu)
file_path = "/content/drive/MyDrive/Semester 7/PPW/pta_all.csv"

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
    ['id', 'penulis', 'judul', 'abstrak_id', 'abstrak_en', 'pembimbing_pertama', 'pembimbing_kedua', 'prodi']
    
    Contoh 100 baris pertama:
                 id                           penulis  \
    0   80111100012               Dyah Ayu Citra Seza   
    1   80111100002                  Maulina Nurlaily   
    2   70111100060               Moh. Samsul Hidayat   
    3   90111100077  TOMMY ADITYA PARLINDUNGAN MARBUN   
    4   70111200007                RICA YENA IMADHORA   
    ..          ...                               ...   
    95  90111100078             Pradana Anggara Murti   
    96  80111100065                               NaN   
    97  70111100030                           ERFANDI   
    98  80111100024               RIDLO ERFIN SANTOSO   
    99  70111200008                               NaN   
    
                                                    judul  \
    0   Implementasi Fungsi Legislasi Dewan Perwakilan...   
    1   Pertanggungjawaban Pidana Direksi BUMN (Perser...   
    2   Analisis Terhadap Kekosongan Hukum dalam Penga...   
    3   PERLINDUNGAN HUKUM BAGI KONSUMEN ATAS PRODUK E...   
    4   TELAAH  KRITIS TENTANG ALASAN HUKUM YANG DIGUN...   
    ..                                                ...   
    95  PENGANGKATAN HAKIM KONSTITUSI MENURUT \r\nPERA...   
    96  UPAYA BALAI KONSERVASI SUMBER DAYA ALAM DALAM ...   
    97  KEDUDUKAN KETERANGAN TESTIMONIUM DE AUDITU DAL...   
    98  PENUNTUTAN PIDANA TERHADAP PERBUATAN MEMBOCORK...   
    99  PERSPEKTIF HUKUM TURUT SERTA MENYEBARKAN RAHAS...   
    
                                               abstrak_id  \
    0   ABSTRAK\r\n\r\n       Implementasi Fungsi Legi...   
    1   Badan Usaha Milik Negara (BUMN) adalah Badan u...   
    2   Kasus narkoba tidak henti-hentinya terdengar d...   
    3   Produk elektronik adalah suatu benda bergerak ...   
    4                                                 NaN   
    ..                                                ...   
    95  ABSTRAK\r\nHakim Konstitusi merupakan salah sa...   
    96  ABSTRAK\r\n\r\nAktivitas perdagangan satwa lia...   
    97  ABSTRAK\r\n\r\nKeterangan testimonium de aidit...   
    98  ABSTRAK\r\nPerbuatan membocorkan dokumen Surat...   
    99  ABSTRAK\r\nPerkembangan yang pesat di bidang t...   
    
                                               abstrak_en  \
    0   ABSTRACT\r\n       Implementation of Legislati...   
    1   State Owned Enterprises (SOEs) are business en...   
    2   Drug cases endlessly heard on television, radi...   
    3   Electronic products is an object moves through...   
    4                                                 NaN   
    ..                                                ...   
    95  ABSTRACT\r\nConstitutional judges is one of th...   
    96  ABSTRACT\r\n\r\nActivity wildlife trade end - ...   
    97  ABSTRACT\r\n\r\nDescription testimonium de aid...   
    98  The act of leaking documents Warrant Investiga...   
    99  ABSTRACT\r\nThe rapid developments in the fiel...   
    
                              pembimbing_pertama  \
    0            Yudi Widagdo Harimurti, SH., MH   
    1                    Tolib Effendi, SH., MH.   
    2                    Tolib Effendi, SH., MH.   
    3                  DR. DJULAEKA, S.H., M.HUM   
    4                 Dr. DENI SBY, S. H., M. S.   
    ..                                       ...   
    95                Dr.Nunuk Nuswardani,SH,.MH   
    96                DR. WARTININGSIH SH.,M.Hum   
    97                Dr. Eny Suastuti, SH.,MHum   
    98  Dr.DENI SETYA BAGUS YUHERAWAN.,S.H., M.S   
    99                  Tolib Effendi, S.H., M.H   
    
                        pembimbing_kedua       prodi  
    0                     Safi', SH., MH  Ilmu Hukum  
    1       Dr. Eni Suastuti, SH., Mhum.  Ilmu Hukum  
    2            Agus Ramdlany, SH., MH.  Ilmu Hukum  
    3   DR.USWATUN HASANAH, S.H., M. HUM  Ilmu Hukum  
    4      SAIFUL ABDULLAH, S. H., M. H.  Ilmu Hukum  
    ..                               ...         ...  
    95    Encik Muhammad Fauzan,SH,.LL.M  Ilmu Hukum  
    96            TOLIB EFFENDI, SH., MH  Ilmu Hukum  
    97    Dr. Deni Setya Bagus Y. SH.,MS  Ilmu Hukum  
    98     Dr. ENY SUASTUTI.,S.H., M.Hum  Ilmu Hukum  
    99    Rusmilawati Windari, S.H., M.H  Ilmu Hukum  
    
    [100 rows x 8 columns]
    

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
df['preprocessed'] = df['abstrak_id'].astype(str).apply(preprocess)
```


```python
import re
import pandas as pd
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory

# Buat stemmer Sastrawi
factory = StemmerFactory()
stemmer = factory.create_stemmer()

# Ambil 5 data awal
sample_data = df['abstrak_id'].astype(str).head(5)

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



  <div id="df-4e079abe-1a5c-4f27-86f2-d9641af7507f" class="colab-df-container">
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
      <td>ABSTRAK\r\n\r\n       Implementasi Fungsi Legi...</td>
      <td>[ABSTRAK, Implementasi, Fungsi, Legislasi, DPR...</td>
      <td>[ABSTRAK, Implementasi, Fungsi, Legislasi, DPR...</td>
      <td>[abstrak, implementasi, fungsi, legislasi, dpr...</td>
      <td>[abstrak, implementasi, fungsi, legislasi, dpr...</td>
      <td>[abstrak, implementasi, fungsi, legislasi, dpr...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Badan Usaha Milik Negara (BUMN) adalah Badan u...</td>
      <td>[Badan, Usaha, Milik, Negara, (BUMN), Badan, u...</td>
      <td>[Badan, Usaha, Milik, Negara, BUMN, Badan, usa...</td>
      <td>[badan, usaha, milik, negara, bumn, badan, usa...</td>
      <td>[badan, usaha, milik, negara, bumn, badan, usa...</td>
      <td>[badan, usaha, milik, negara, bumn, badan, usa...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Kasus narkoba tidak henti-hentinya terdengar d...</td>
      <td>[Kasus, narkoba, henti-hentinya, terdengar, me...</td>
      <td>[Kasus, narkoba, hentihentinya, terdengar, med...</td>
      <td>[kasus, narkoba, hentihentinya, terdengar, med...</td>
      <td>[kasus, narkoba, hentihentinya, dengar, media,...</td>
      <td>[kasus, narkoba, hentihentinya, dengar, media,...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Produk elektronik adalah suatu benda bergerak ...</td>
      <td>[Produk, elektronik, suatu, benda, bergerak, d...</td>
      <td>[Produk, elektronik, suatu, benda, bergerak, d...</td>
      <td>[produk, elektronik, suatu, benda, bergerak, d...</td>
      <td>[produk, elektronik, suatu, benda, gerak, hasi...</td>
      <td>[produk, elektronik, suatu, benda, gerak, hasi...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>nan</td>
      <td>[nan]</td>
      <td>[nan]</td>
      <td>[nan]</td>
      <td>[nan]</td>
      <td>[nan]</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-4e079abe-1a5c-4f27-86f2-d9641af7507f')"
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
        document.querySelector('#df-4e079abe-1a5c-4f27-86f2-d9641af7507f button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-4e079abe-1a5c-4f27-86f2-d9641af7507f');
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


    <div id="df-3e95a7b6-4f52-4465-8621-2587dd112ea0">
      <button class="colab-df-quickchart" onclick="quickchart('df-3e95a7b6-4f52-4465-8621-2587dd112ea0')"
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
            document.querySelector('#df-3e95a7b6-4f52-4465-8621-2587dd112ea0 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

  <div id="id_562dfb60-d3e3-4658-90f9-244f87ce387f">
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
        document.querySelector('#id_562dfb60-d3e3-4658-90f9-244f87ce387f button.colab-df-generate');
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
sample_data = df['abstrak_id'].astype(str)

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
save_path = "/content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing.csv"

# Simpan otomatis ke Drive
tahapan_df.to_csv(save_path, index=False, encoding='utf-8-sig')

print(f"✅ File berhasil disimpan di {save_path}")

```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    ✅ File berhasil disimpan di /content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing.csv
    
