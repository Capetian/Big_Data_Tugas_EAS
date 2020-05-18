Kelas: Big Data

Nama: Raden Bimo Rizki Prayogo

NRP: 0511740000139

# Big Data - Tugas EAS

# Daily minimum temperatures
## Business Understanding
Workflow ini digunakan untuk melakukan analisa time series terhadap temperatur minimum setip harinya pada suatu tempat

Overview Workflow:

![picture](/img/overview.PNG)

## Data Understanding

Data yang digunakan adalah dataset CSV daily-minimum-temperature-in-me. 

Dataset mengandung 3650 baris yang berisi informasi tentang penggunaan listrik suatu pada suatu meteran pada suatu saat. Dataset mengandung 2 kolom yakni:

- Date, tanggal temperatur dicatat
- Daily minimum temperatures, temperatur minimum pada hari tersebut



![picture](/img/dataset.PNG)


## Data Preparation
### Membuat Spark Context
Pertama-tama sebuah context spark perlu disiapkan terlebih dahulu. Spark Context dibuat secara lokal dan menggunakan 2 executor (thread).

![picture](/img/spark_setup.PNG)

![picture](/img/spark_con.PNG)

### Membaca dataset

![picture](/img/read_spark1.PNG)

Isi metanode load data.

![picture](/img/read_spark2.PNG)

Dataset pada csv dibaca dengan file reader. Dataset tersebut dijadikan tabel pada suatu koneksi Hive terlebih dahulu di metanode Load Data. Table Hive yang dihasilkan dijadikan sebuah spark Dataframe dengan node Hive to Spark.

Dataframe yang dihasilkan:

![picture](/img/res_read.PNG)

### Mengektraksi time series

![picture](/img/extraction1.PNG)

Isi Extract date-time attributes:

![picture](/img/extraction2.PNG)


Di metanode Extract date-time attributes kita melakukan konversi kolom Date menjadi tanggal dan kolom Daily minimum temperatures menjadi double. Dari kolom Date yang dijadikan tanggal, kita mengektraksi tahun, kuarter tahun, bulan, minggu, dan hari pada minggu. Untuk menghandle jika ada row dengan nilai yang hilang akibat extraksi data, kita teruskan ke node Spark Missing Value.

Query ekstraksi tanggal dan waktu:

![picture](/img/extraction_date_time.PNG)

Hasil ekstraksi tanggal dan waktu:

![picture](/img/extraction_date_time_res.PNG)

Dari tabel yang dihasilkan pada query sebelumnya, kita mengektraksi tahun, kuarter tahun, bulan, minggu, dan hari pada minggu dengan query berikut:

![picture](/img/extraction_from_date_time.PNG)

Hasil query:

![picture](/img/extraction_from_date_time_res.PNG)

kolom tahun sebelumnya tidak di cast sebagai angka agar dianggap sebagai data kategori, ubah kategori menjadi angka dengan node Spark Category to Number. 

![picture](/img/extraction_from_year_res.PNG)

Untuk menghandle missing value kita gunakan node Spark Missing Value.

![picture](/img/extraction_missing.PNG)

Hasil ekstraksi:

![picture](/img/extraction_res.PNG)


## Modelling

![picture](/img/modelling1.PNG)


Kita melakukan dua macam analisis, dengan metanode Predictions kita mengubah dataset time series yang memiliki seasonality menjadi dataset time series yang stasioner, kemudian kita membuat model untuk melalukan prediksi dengan menggunakan partisi dari dataset yang ada. Analisis yang dilakukan pada metanode Cluster Season yakni melakukan cluster berdasarkan musim menggunakan kolom hasil ekstraksi waktu.

### Prediksi dengan metanode Predictions

Isi metanode Predictions:

![picture](/img/modelling2.PNG)

Untuk mengubah time series menjadi time series stasioner, kita dapat melakukan first order difference, yakni mengurangi nilai ke-t dari suatu time series dengan value ke t-1. Pada kasus ini kita melakukan first order difference pada nilai temperatur. Pertama-tama kita perlu melakukan lag terlebih dahulu pada kolom temperatur.

Query lag:

![picture](/img/modelling1_lag.PNG)

Hasil query:

![picture](/img/modelling1_lag_res.PNG)


Hasil query menghasilkan missing value pada row pertama karena tidak ada nilai t-1 yang dapat diambil. Oleh karena itu kita perlu menghandle missing value tersebut dengan Spark Missing Value.

Node missing value:

![picture](/img/modelling1_missing.PNG)

Hasil node:

![picture](/img/modelling1_missing_res.PNG)


Kemudian kita lakukan first order difference dengan query berikut:

![picture](/img/modelling1_first_ord.PNG)

Hasil query:

![picture](/img/modelling1_first_ord_res.PNG)

Kita kemudian melakukan partisi training dan test dengan Spark Partition dan membuat model prediksi temperatur dengan Spark Gradient Boosted Trees Learner. Spark Gradient Boosted Trees Learner dipilih karena menghasilkan error yang paling sedikit dibanding tipe ML yang lain, namun sayangnya KNIME tidak bisa mengubah tipe model tersebut menjadi PMML (Spark Gradient Boosted Trees versi MLlib spark juga tidak bisa dijadikan PMML dengan node Spark MLlib to PMML). Model ditraining dengan partisi training menggunakan temperatur sbelum lag/ temperatur asli sebagai label dan hasil ektraksi tanggal, first order difference dan temperatur lag sebagai fitur. Setelah itu, kita melakukan prediksi dengan Spark Predictor pada partisi test.

Konfigurasi partisi:

![picture](/img/modelling1_partition.PNG)

Konfigurasi model:

![picture](/img/modelling1_model.PNG)

Hasil Prediksi:

![picture](/img/modelling1_model_res.PNG)


### Clustering Musim dengan metanode Cluster Seasons

Isi metanode Cluster Seasons:

![picture](/img/modelling3.PNG)

Pada metanode Cluster Seasons kita melakukan cluster menjadi 4 cluster yang harapannya akan menggambarkan 4 musim. Kita melakukan cluster dengan Spark K-Means dengan kolom yang diektraksi dari tanggal dan temperatur sebagai fitur.

Konfigurasi K-Means Clustering

![picture](/img/modelling2_conf.PNG)


Hasil Cluster:

![picture](/img/modelling2_res.PNG)


## Evaluasi

### Evaluasi metanode Predictions

![picture](/img/modelling2.PNG)

Model yang dihasilkan dievaluasi dengan Spark Numeric Scorer, selain itu hasil prediksi di-plot di graf untuk memvisualisasikan hasil prediksi dengan ground truth.

Hasil Scoring:

![picture](/img/evaluation1_score.PNG)

Membuat grafik tersebut, kita perlu menggabungkan partisi terlebih dahulu, kemudian mengubah spark dataframe menjadi tabel.

Query untuk menambahkan kolom prediksi ke partisi pertama:

![picture](/img/evaluation1_add_pred.PNG)

Penggabungan partisi dengan Spark Concatenate:

![picture](/img/evaluation1_concate.PNG)

Mengubah dataframe menjadi tabel dengan Spark to Table:

![picture](/img/evaluation1_to_table.PNG)

Grafik yang dihasilkan:

![picture](/img/evaluation1_graph.PNG)

### Evaluasi metanode Cluster Seasons

![picture](/img/modelling3.PNG)

Data yang dihasilkan diwarnai berdasarkan cluster dan digambar di suatu grafik dimana x adalah tanggal dan y adalah cluster. 

Pewarnaan Cluster:

![picture](/img/evaluation2_colour.PNG)

Grafik yang dihasilkan:

![picture](/img/evaluation2_graph.PNG)


## Deployment 

![picture](/img/deployment.PNG)

Hasil prediksi dan cluster yang dibentuk sebelumnya di deploy ke Hive dalam bentuk Paraquet.

Konfigurasi deployment prediksi: 

![picture](/img/deployment1_conf.PNG)

Konfigurasi deployment cluster: 

![picture](/img/deployment2_conf.PNG)

Hasil deployment prediksi: 

![picture](/img/deployment1.PNG)

Konfigurasi deployment cluster: 

![picture](/img/deployment2.PNG)
