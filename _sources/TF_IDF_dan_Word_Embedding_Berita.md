## TF-IDF dan Word Embeding

**TF-IDF**


```python
!pip install scikit-learn
```

    Requirement already satisfied: scikit-learn in /usr/local/lib/python3.12/dist-packages (1.6.1)
    Requirement already satisfied: numpy>=1.19.5 in /usr/local/lib/python3.12/dist-packages (from scikit-learn) (2.0.2)
    Requirement already satisfied: scipy>=1.6.0 in /usr/local/lib/python3.12/dist-packages (from scikit-learn) (1.16.2)
    Requirement already satisfied: joblib>=1.2.0 in /usr/local/lib/python3.12/dist-packages (from scikit-learn) (1.5.2)
    Requirement already satisfied: threadpoolctl>=3.1.0 in /usr/local/lib/python3.12/dist-packages (from scikit-learn) (3.6.0)
    


```python
import pandas as pd
import ast
from sklearn.feature_extraction.text import TfidfVectorizer
```


```python
from google.colab import drive
import pandas as pd

# Mount Google Drive
drive.mount('/content/drive')

```

    Mounted at /content/drive
    


```python
# Baca CSV dari path Drive
df = pd.read_csv('/content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing_berita.csv')
df.head()
```





  <div id="df-1c859443-d57f-4ec6-ae34-94d94f010024" class="colab-df-container">
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
      <th>Teks Asli</th>
      <th>Setelah Stopword Removal</th>
      <th>Setelah Cleaning</th>
      <th>Setelah Normalisasi</th>
      <th>Setelah Stemming</th>
      <th>Tokenisasi Akhir</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alfamart kembali menggelar aksi donor darah se...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'gelar', 'aksi', 'dono...</td>
      <td>['alfamart', 'kembali', 'gelar', 'aksi', 'dono...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>menteri koordinator bidang hukum, ham, imigras...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum,',...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>gubernur sumatera utara (sumut) bobby nasution...</td>
      <td>['gubernur', 'sumatera', 'utara', '(sumut)', '...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dukungan untuk plt ketua umum partai persatuan...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukung', 'plt', 'ketua', 'umum', 'partai', '...</td>
      <td>['dukung', 'plt', 'ketua', 'umum', 'partai', '...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bagi pengguna whoosh rute jakarta-bandung atau...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'guna', 'whoosh', 'rute', 'jakartaban...</td>
      <td>['bagi', 'guna', 'whoosh', 'rute', 'jakartaban...</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-1c859443-d57f-4ec6-ae34-94d94f010024')"
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
        document.querySelector('#df-1c859443-d57f-4ec6-ae34-94d94f010024 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-1c859443-d57f-4ec6-ae34-94d94f010024');
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


    <div id="df-a24d496b-9118-4edc-99d9-359af7f2ff43">
      <button class="colab-df-quickchart" onclick="quickchart('df-a24d496b-9118-4edc-99d9-359af7f2ff43')"
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
            document.querySelector('#df-a24d496b-9118-4edc-99d9-359af7f2ff43 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>





```python
print(type(df["Tokenisasi Akhir"].iloc[0]))
print(df["Tokenisasi Akhir"].iloc[0])
```

    <class 'str'>
    ['alfamart', 'kembali', 'gelar', 'aksi', 'donor', 'darah', 'serentak', 'kotakabupaten', 'lama', 'satu', 'minggu', 'september', 'dalam', 'program', 'sehat', 'tetes', 'darah', 'pertiwi', 'giat', 'sosial', 'ini', 'alfamart', 'target', 'kumpul', 'kantong', 'darah', 'bantu', 'penuh', 'stok', 'palang', 'merah', 'indonesia', 'pmi', 'ini', 'salah', 'satu', 'kontribusi', 'alfamart', 'bantu', 'pasien', 'butuh', 'transfusi', 'darah', 'gandeng', 'pmi', 'daerah', 'harap', 'ribu', 'kantong', 'darah', 'dapat', 'capai', 'salur', 'kepada', 'butuh', 'kata', 'corporate', 'affairs', 'director', 'alfamart', 'solihin', 'dalam', 'terang', 'tulis', 'rabu', 'scroll', 'continue', 'with', 'content', 'giat', 'donor', 'darah', 'kali', 'jadi', 'bagi', 'rangkai', 'semarak', 'ulang', 'tahun', 'alfamart', 'sua', 'ke', 'target', 'kantong', 'darah', 'pilih', 'bagai', 'refleksi', 'usia', 'alfamart', 'telah', 'tahun', 'layan', 'masyarakat', 'indonesia', 'solihin', 'tambah', 'donor', 'darah', 'telah', 'jadi', 'giat', 'rutin', 'alfamart', 'tahun', 'ini', 'program', 'dapat', 'dukung', 'jumlah', 'mitra', 'seperti', 'bear', 'brand', 'milo', 'aquviva', 'ichitan', 'paper', 'bag', 'program', 'jadi', 'bagi', 'tanggung', 'jawab', 'sosial', 'usaha', 'kumpul', 'darah', 'dalam', 'jumlah', 'besar', 'makin', 'banyak', 'pasien', 'bisa', 'bantu', 'ujar', 'solihin', 'selain', 'donor', 'darah', 'giat', 'turut', 'isi', 'talkshow', 'sehat', 'sama', 'direktur', 'rumah', 'sakit', 'islam', 'sari', 'asih', 'arrahmah', 'irhami', 'sempat', 'itu', 'irhami', 'tekan', 'penting', 'siap', 'belum', 'donor', 'darah', 'selain', 'pasti', 'kondisi', 'tubuh', 'sehat', 'calon', 'donor', 'perlu', 'jaga', 'pola', 'tidur', 'cukup', 'serta', 'pola', 'makan', 'atur', 'agar', 'proses', 'donor', 'darah', 'jalan', 'lancar', 'ujar', 'irhami', 'apresiasi', 'khusus', 'beri', 'lalu', 'blood', 'heroes', 'bagi', 'donor', 'telah', 'sumbang', 'darah', 'lebih', 'kali', 'harga', 'serah', 'langsung', 'oleh', 'solihin', 'sama', 'wakil', 'ketua', 'pmi', 'provinsi', 'banten', 'jaenudin', 'sementara', 'itu', 'jaenudin', 'sampai', 'apresiasi', 'hadap', 'program', 'donor', 'darah', 'jalan', 'alfamart', 'kumpul', 'kantong', 'darah', 'kumpul', 'lama', 'september', 'alfamart', 'dapat', 'jadi', 'contoh', 'bagi', 'usaha', 'lain', 'donor', 'darah', 'bukan', 'hanya', 'manfaat', 'bagi', 'sehat', 'tetapi', 'wujud', 'nyata', 'peduli', 'kepada', 'sama', 'ujar', 'jaenudin', 'kantong', 'darah', 'kumpul', 'akan', 'sangat', 'arti', 'bagi', 'hidup', 'orang', 'lain', 'gantung', 'kepada', 'stok', 'darah', 'pmi', 'sambung', 'para', 'serta', 'donor', 'darah', 'dapat', 'paket', 'makan', 'gizi', 'goodie', 'bag', 'bagai', 'tambah', 'nutrisi', 'seperti', 'susu', 'murni', 'bear', 'brand', 'susu', 'sapi', 'steril', 'murni', 'milo', 'biskuit', 'air', 'mineral', 'aquviva', 'telah', 'lalu', 'tahap', 'nano', 'purifikasi', 'pasti', 'murni', 'imbang', 'mineral', 'ichitan', 'serta', 'pelbagai', 'nutrisi', 'tambah', 'salah', 'orang', 'donor', 'baru', 'pertama', 'kali', 'laku', 'donor', 'darah', 'rico', 'aku', 'awal', 'sempat', 'takut', 'sedikit', 'takut', 'awal', 'jarum', 'tapi', 'rani', 'diri', 'harap', 'darah', 'bisa', 'manfaat', 'kata', 'rico', 'donor', 'lain', 'mas', 'justru', 'rutin', 'ikut', 'giat', 'ini', 'manfaat', 'rasa', 'langsung', 'apalagi', 'bantu', 'butuh', 'alfamart', 'konsisten', 'ada', 'ujar', 'mas', 'giat', 'donor', 'darah', 'alfamart', 'jadi', 'bagi', 'upaya', 'dukung', 'sedia', 'darah', 'pmi', 'sangat', 'butuh', 'pasien', 'masuk', 'idap', 'thalassemia', 'seperti', 'intan', 'turut', 'beri', 'cerita', 'acara', 'sebut', 'intan', 'harus', 'rutin', 'jalan', 'transfusi', 'darah', 'sejak', 'usia', 'tahun', 'bagi', 'intan', 'tiap', 'tetes', 'darah', 'sedia', 'pmi', 'topang', 'hidup', 'kalau', 'transfusi', 'badan', 'lemas', 'sekali', 'rasa', 'sanggup', 'diri', 'donor', 'darah', 'seperti', 'ini', 'bisa', 'tetap', 'lanjut', 'aktivitas', 'seharihari', 'tutur', 'intan', 'kisah', 'intan', 'jadi', 'bukti', 'nyata', 'bahwa', 'donor', 'darah', 'sekadar', 'angka', 'tetapi', 'tentang', 'harap', 'selamat', 'hidup']
    


```python
df["Tokenisasi Akhir_joined"] = df["Tokenisasi Akhir"].apply(
    lambda x: " ".join(ast.literal_eval(x)) if isinstance(x, str) else ""
)


df[["Tokenisasi Akhir", "Tokenisasi Akhir_joined"]].head()
```





  <div id="df-6b926698-97f1-4180-8035-1dbec035b35c" class="colab-df-container">
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
      <th>Tokenisasi Akhir</th>
      <th>Tokenisasi Akhir_joined</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>['alfamart', 'kembali', 'gelar', 'aksi', 'dono...</td>
      <td>alfamart kembali gelar aksi donor darah serent...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>menteri koordinator bidang hukum ham imigrasi ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>gubernur sumatera utara sumut bobby nasution t...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>['dukung', 'plt', 'ketua', 'umum', 'partai', '...</td>
      <td>dukung plt ketua umum partai satu bangun ppp m...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>['bagi', 'guna', 'whoosh', 'rute', 'jakartaban...</td>
      <td>bagi guna whoosh rute jakartabandung atau bali...</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-6b926698-97f1-4180-8035-1dbec035b35c')"
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
        document.querySelector('#df-6b926698-97f1-4180-8035-1dbec035b35c button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-6b926698-97f1-4180-8035-1dbec035b35c');
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


    <div id="df-7853af1d-9a06-4731-b24b-09a6f90fcd61">
      <button class="colab-df-quickchart" onclick="quickchart('df-7853af1d-9a06-4731-b24b-09a6f90fcd61')"
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
            document.querySelector('#df-7853af1d-9a06-4731-b24b-09a6f90fcd61 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>





```python
vectorizer = TfidfVectorizer(max_features=1000)  # ambil 1000 fitur teratas

X_tfidf = vectorizer.fit_transform(df["Tokenisasi Akhir_joined"])

tfidf_df = pd.DataFrame(
    X_tfidf.toarray(),
    columns=vectorizer.get_feature_names_out()
)

tfidf_df.head()
```





  <div id="df-b9513b2a-647a-485e-8005-f030b53f53f0" class="colab-df-container">
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
      <th>acara</th>
      <th>ada</th>
      <th>adapun</th>
      <th>adil</th>
      <th>afp</th>
      <th>afriansyah</th>
      <th>agam</th>
      <th>agama</th>
      <th>agar</th>
      <th>agenda</th>
      <th>...</th>
      <th>wira</th>
      <th>with</th>
      <th>wujud</th>
      <th>xiii</th>
      <th>ya</th>
      <th>yaitu</th>
      <th>yakin</th>
      <th>yakni</th>
      <th>yang</th>
      <th>yayasan</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.016249</td>
      <td>0.010689</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.013411</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.005929</td>
      <td>0.018111</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.012363</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.0</td>
      <td>0.035928</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.014949</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.00000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.033599</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.014854</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.04709</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.029843</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.079639</td>
      <td>0.00000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 1000 columns</p>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-b9513b2a-647a-485e-8005-f030b53f53f0')"
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
        document.querySelector('#df-b9513b2a-647a-485e-8005-f030b53f53f0 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-b9513b2a-647a-485e-8005-f030b53f53f0');
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


    <div id="df-fa01f519-292f-48d8-b9aa-ca92cbc24d1d">
      <button class="colab-df-quickchart" onclick="quickchart('df-fa01f519-292f-48d8-b9aa-ca92cbc24d1d')"
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
            document.querySelector('#df-fa01f519-292f-48d8-b9aa-ca92cbc24d1d button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>




**Word Embedding**


```python
!pip install gensim
```

    Collecting gensim
      Downloading gensim-4.3.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (8.1 kB)
    Collecting numpy<2.0,>=1.18.5 (from gensim)
      Downloading numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m61.0/61.0 kB[0m [31m5.5 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting scipy<1.14.0,>=1.7.0 (from gensim)
      Downloading scipy-1.13.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m60.6/60.6 kB[0m [31m5.5 MB/s[0m eta [36m0:00:00[0m
    [?25hRequirement already satisfied: smart-open>=1.8.1 in /usr/local/lib/python3.12/dist-packages (from gensim) (7.3.1)
    Requirement already satisfied: wrapt in /usr/local/lib/python3.12/dist-packages (from smart-open>=1.8.1->gensim) (1.17.3)
    Downloading gensim-4.3.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26.6 MB)
    [2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m26.6/26.6 MB[0m [31m71.2 MB/s[0m eta [36m0:00:00[0m
    [?25hDownloading numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (18.0 MB)
    [2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m18.0/18.0 MB[0m [31m103.0 MB/s[0m eta [36m0:00:00[0m
    [?25hDownloading scipy-1.13.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (38.2 MB)
    [2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m38.2/38.2 MB[0m [31m17.3 MB/s[0m eta [36m0:00:00[0m
    [?25hInstalling collected packages: numpy, scipy, gensim
      Attempting uninstall: numpy
        Found existing installation: numpy 2.0.2
        Uninstalling numpy-2.0.2:
          Successfully uninstalled numpy-2.0.2
      Attempting uninstall: scipy
        Found existing installation: scipy 1.16.2
        Uninstalling scipy-1.16.2:
          Successfully uninstalled scipy-1.16.2
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    opencv-python 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    tsfresh 0.21.1 requires scipy>=1.14.0; python_version >= "3.10", but you have scipy 1.13.1 which is incompatible.
    opencv-python-headless 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    opencv-contrib-python 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    thinc 8.3.6 requires numpy<3.0.0,>=2.0.0, but you have numpy 1.26.4 which is incompatible.[0m[31m
    [0mSuccessfully installed gensim-4.3.3 numpy-1.26.4 scipy-1.13.1
    




```python
!pip install --upgrade --force-reinstall numpy gensim
```

    Collecting numpy
      Downloading numpy-2.3.3-cp312-cp312-manylinux_2_27_x86_64.manylinux_2_28_x86_64.whl.metadata (62 kB)
    [?25l     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m0.0/62.1 kB[0m [31m?[0m eta [36m-:--:--[0m
[2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m62.1/62.1 kB[0m [31m3.4 MB/s[0m eta [36m0:00:00[0m
    [?25hCollecting gensim
      Using cached gensim-4.3.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (8.1 kB)
    Collecting numpy
      Using cached numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (61 kB)
    Collecting scipy<1.14.0,>=1.7.0 (from gensim)
      Using cached scipy-1.13.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (60 kB)
    Collecting smart-open>=1.8.1 (from gensim)
      Downloading smart_open-7.3.1-py3-none-any.whl.metadata (24 kB)
    Collecting wrapt (from smart-open>=1.8.1->gensim)
      Downloading wrapt-1.17.3-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (6.4 kB)
    Using cached gensim-4.3.3-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (26.6 MB)
    Using cached numpy-1.26.4-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (18.0 MB)
    Using cached scipy-1.13.1-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (38.2 MB)
    Downloading smart_open-7.3.1-py3-none-any.whl (61 kB)
    [2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m61.7/61.7 kB[0m [31m5.0 MB/s[0m eta [36m0:00:00[0m
    [?25hDownloading wrapt-1.17.3-cp312-cp312-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (88 kB)
    [2K   [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m88.0/88.0 kB[0m [31m7.8 MB/s[0m eta [36m0:00:00[0m
    [?25hInstalling collected packages: wrapt, numpy, smart-open, scipy, gensim
      Attempting uninstall: wrapt
        Found existing installation: wrapt 1.17.3
        Uninstalling wrapt-1.17.3:
          Successfully uninstalled wrapt-1.17.3
      Attempting uninstall: numpy
        Found existing installation: numpy 1.26.4
        Uninstalling numpy-1.26.4:
          Successfully uninstalled numpy-1.26.4
      Attempting uninstall: smart-open
        Found existing installation: smart_open 7.3.1
        Uninstalling smart_open-7.3.1:
          Successfully uninstalled smart_open-7.3.1
      Attempting uninstall: scipy
        Found existing installation: scipy 1.13.1
        Uninstalling scipy-1.13.1:
          Successfully uninstalled scipy-1.13.1
      Attempting uninstall: gensim
        Found existing installation: gensim 4.3.3
        Uninstalling gensim-4.3.3:
          Successfully uninstalled gensim-4.3.3
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    opencv-python 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    tsfresh 0.21.1 requires scipy>=1.14.0; python_version >= "3.10", but you have scipy 1.13.1 which is incompatible.
    opencv-python-headless 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    opencv-contrib-python 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
    thinc 8.3.6 requires numpy<3.0.0,>=2.0.0, but you have numpy 1.26.4 which is incompatible.[0m[31m
    [0mSuccessfully installed gensim-4.3.3 numpy-1.26.4 scipy-1.13.1 smart-open-7.3.1 wrapt-1.17.3
    




```python
import pandas as pd
import ast
from gensim.models import Word2Vec
```


```python
from google.colab import drive
import pandas as pd

# Mount Google Drive
drive.mount('/content/drive')

```

    Drive already mounted at /content/drive; to attempt to forcibly remount, call drive.mount("/content/drive", force_remount=True).
    


```python
# Baca CSV dari path Drive
df = pd.read_csv('/content/drive/MyDrive/Semester 7/PPW/hasil_preprocessing_berita.csv')
df.head()
```





  <div id="df-90608a75-1f8c-450d-98fa-6ec33dd260ef" class="colab-df-container">
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
      <th>Teks Asli</th>
      <th>Setelah Stopword Removal</th>
      <th>Setelah Cleaning</th>
      <th>Setelah Normalisasi</th>
      <th>Setelah Stemming</th>
      <th>Tokenisasi Akhir</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>alfamart kembali menggelar aksi donor darah se...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'menggelar', 'aksi', '...</td>
      <td>['alfamart', 'kembali', 'gelar', 'aksi', 'dono...</td>
      <td>['alfamart', 'kembali', 'gelar', 'aksi', 'dono...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>menteri koordinator bidang hukum, ham, imigras...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum,',...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
      <td>['menteri', 'koordinator', 'bidang', 'hukum', ...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>gubernur sumatera utara (sumut) bobby nasution...</td>
      <td>['gubernur', 'sumatera', 'utara', '(sumut)', '...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
      <td>['gubernur', 'sumatera', 'utara', 'sumut', 'bo...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dukungan untuk plt ketua umum partai persatuan...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukungan', 'plt', 'ketua', 'umum', 'partai',...</td>
      <td>['dukung', 'plt', 'ketua', 'umum', 'partai', '...</td>
      <td>['dukung', 'plt', 'ketua', 'umum', 'partai', '...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bagi pengguna whoosh rute jakarta-bandung atau...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'pengguna', 'whoosh', 'rute', 'jakart...</td>
      <td>['bagi', 'guna', 'whoosh', 'rute', 'jakartaban...</td>
      <td>['bagi', 'guna', 'whoosh', 'rute', 'jakartaban...</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-90608a75-1f8c-450d-98fa-6ec33dd260ef')"
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
        document.querySelector('#df-90608a75-1f8c-450d-98fa-6ec33dd260ef button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-90608a75-1f8c-450d-98fa-6ec33dd260ef');
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


    <div id="df-b7cf64f3-c1db-4763-a8e2-ae5ac69fd397">
      <button class="colab-df-quickchart" onclick="quickchart('df-b7cf64f3-c1db-4763-a8e2-ae5ac69fd397')"
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
            document.querySelector('#df-b7cf64f3-c1db-4763-a8e2-ae5ac69fd397 button');
          quickchartButtonEl.style.display =
            google.colab.kernel.accessAllowed ? 'block' : 'none';
        })();
      </script>
    </div>

    </div>
  </div>





```python
corpus = df["Tokenisasi Akhir"].apply(
    lambda x: ast.literal_eval(x) if isinstance(x, str) else []
).tolist()
```


```python
model = Word2Vec(
    sentences=corpus,
    vector_size=100,   # dimensi vektor
    window=5,          # konteks window
    min_count=2,       # kata muncul minimal 2 kali
    sg=1,              # 1=skip-gram, 0=CBOW
    workers=4
)
```


```python
model.save("word2vec_berita.model")
```


```python
# === Cek hasil ===
print("\nVektor untuk kata 'indonesia':")
print(model.wv['indonesia'][:10])

print("\nKata yang mirip dengan 'indonesia':")
print(model.wv.most_similar("indonesia", topn=5))
```

    
    Vektor untuk kata 'indonesia':
    [-0.11810647  0.17236815  0.00422871 -0.10403389  0.0705354  -0.35762382
      0.0828401   0.3290212  -0.1874316  -0.02492193]
    
    Kata yang mirip dengan 'indonesia':
    [('republik', 0.9742425084114075), ('selamat', 0.9715856313705444), ('prioritas', 0.9707692265510559), ('segala', 0.9696119427680969), ('atur', 0.9682633876800537)]
    
