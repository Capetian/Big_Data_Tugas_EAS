Kelas: Big Data

Nama: Raden Bimo Rizki Prayogo

NRP: 0511740000139

# Big Data - Tugas 7

# Monthly Beer Production
## Business Understanding
Workflow ini digunakan untuk melakukan analisa time series terhadap produksi bir setiap bulannya.  Analisa dilakukan dalam dua bentuk yakni, pembentukkan cluster dan prediksi nilai produksi bir.

Overview Workflow:

![picture](/monthly_beer_production/img/overview.PNG)

## Data Understanding

Data yang digunakan adalah dataset CSV monthly beer production. 

Dataset mengandung 476 baris yang berisi informasi tentang produksi bir suatu bulan. Dataset mengandung 2 kolom yakni:

- Month, yakni waktu perekaman produksi bir
- Monthly beer production, produksi bir pada bulan tersebut


![picture](/monthly_beer_production/img/dataset.PNG)


## Data Preparation
### Membuat Spark Context
Pertama-tama sebuah context spark perlu disiapkan terlebih dahulu. Spark Context dibuat secara lokal dan menggunakan 2 executor (thread).

![picture](/monthly_beer_production/img/spark_setup.PNG)

![picture](/monthly_beer_production/img/spark_con.PNG)

### Membaca dataset

![picture](/monthly_beer_production/img/read_spark.PNG)

Isi metanode load data.

![picture](/monthly_beer_production/img/read_spark1.PNG)

Dataset pada csv dibaca dengan file reader. Dataset tersebut dijadikan tabel pada suatu koneksi Hive terlebih dahulu di metanode Load Data. Table Hive yang dihasilkan dijadikan sebuah spark Dataframe dengan node Hive to Spark.

Dataframe yang dihasilkan:

![picture](/monthly_beer_production/img/res_read.PNG)

### Mengektraksi time series

![picture](/monthly_beer_production/img/extraction.PNG)

Isi Extract date-time attributes:

![picture](/monthly_beer_production/img/extraction1.PNG)

Di metanode Extract date-time attributes kita melakukan konversi kolom Month menjadi tanggal.  Dari kolom month yang dijadikan tanggal, kita mengektraksi tahun, kuarter tahun, bulan, minggu, dan hari pada minggu. Untuk menghandle jika ada row dengan nilai yang hilang akibat extraksi data, kita teruskan ke node Spark Missing Value.


Query ekstraksi tanggal dan waktu:

![picture](/monthly_beer_production/img/extraction_date_time.PNG)

Hasil ekstraksi tanggal dan waktu:

![picture](/monthly_beer_production/img/extraction_date_time_res.PNG)

Dari tabel yang dihasilkan pada query sebelumnya, kita mengektraksi tahun, kuarter tahun, bulan, minggu, dan hari pada minggu dengan query berikut:

![picture](/monthly_beer_production/img/extraction_from_date_time.PNG)

Hasil query:

![picture](/monthly_beer_production/img/extraction_from_date_time_res.PNG)

kolom tahun sebelumnya tidak di cast sebagai angka agar dianggap sebagai data kategori, ubah kategori menjadi angka dengan node Spark Category to Number. 

![picture](/monthly_beer_production/img/extraction_from_year_res.PNG)

Untuk menghandle missing value kita gunakan node Spark Missing Value.

![picture](/monthly_beer_production/img/extraction_missing.PNG)

Hasil ekstraksi:

![picture](/monthly_beer_production/img/extraction_res.PNG)

### Mengaggregasi time series

![picture](/monthly_beer_production/img/aggregation.PNG)

Isi metanode Aggregations and time series:

![picture](/monthly_beer_production/img/aggregation1.PNG)


Di metanode Aggregations and time series, pertama-tama kita persist/cache dataframe yang dihasilkan di metanode sebelumnya ke memory untuk mempermudah dan mempercepat aggregasi data. Kemudian kita melakukan aggregasi berupa average terhadap setiap fitur-fitur yang diektraksi sebelumnya. Hasil-hasil agreggasi direname untuk mempermudah pembacaan fitur dan kemudian saling di-join untuk menghasilkan satu tabel/dataframe yang baru dengan hasil-hasil aggregasi.

Melakukan persitence ke memory:

![picture](/monthly_beer_production/img/aggregation_persist.PNG)

Contoh aggregasi fitur:

![picture](/monthly_beer_production/img/aggregation_example.PNG)

Melakukan group by berdasarkan fitur dan lakukan average:

![picture](/monthly_beer_production/img/aggregation_example_group_avg1.PNG)

![picture](/monthly_beer_production/img/aggregation_example_group_avg2.PNG)

Melakukan rename terhadap kolom agreggasi:

![picture](/monthly_beer_production/img/aggregation_example_rename.PNG)

Melakukan join:

![picture](/monthly_beer_production/img/aggregation_join.PNG)


Dataframe yang dihasilkan:

![picture](/monthly_beer_production/img/aggregation_res.PNG)

Filter nilai yang hilang dengan Spark Missing Value:

![picture](/monthly_beer_production/img/aggregation_missing.PNG)


Kita kemudian mendapatkan persentase produksi bir tiap segmen dengan query berikut:

![picture](/monthly_beer_production/img/aggregation_percent.PNG)

Persentase produksi bir tiap segmen didapat dengan membagi segmen dengan fitur yang terbagi oleh segmen tersebut. Contohnya rata-rata pengunaan segmen hari senin dibagi rata-rata pengunaan tiap minggu.

Hasil query:

![picture](/monthly_beer_production/img/aggregation_percent_res.PNG)


### Preprocess time series untuk prediksi


![picture](/monthly_beer_production/img/preprocess.PNG)

Isi metanode preprocess prediction:

![picture](/monthly_beer_production/img/preprocess1.PNG)


Preprocess input dilakukan untuk meningkatkan akurasi prediksi time series. Untuk meningkatkan akurasi prediksi, kita ubah nilai produksi menjadi suatu Moving Average (MA). Window yang digunakan untuk MA sebesar 12 ke belakang, untuk melakukan smoothing dalam skala tahunan. Untuk meningkatkan kualitas time series, kita dapat melakukan first order difference dan menghilangkan seasonality. First order yakni mengurangi nilai ke-t dari suatu time series dengan value ke t-1, sedangkan penghilangan seasonality dilakukan dengan mengurangi nilai ke-t dari suatu time series dengan nilai ke t-periode suatu sifat seasonalitas. Pada kasus ini kita melakukan first order difference pada nilai MA produksi dan penghilangan seasonality tengah tahun (6 bulan). Pertama-tama kita perlu membentuk kolom Moving Average.

Query MA:

![picture](/monthly_beer_production/img/preprocess_MA.PNG)


Hasil Query:

![picture](/monthly_beer_production/img/preprocess_MA.PNG)

Kemudian kita perlu melakukan lag terhadap MA sebanyak 1 untuk first order difference dan 6 untuk penghilangan seasonality pertengahan tahun.

Query lag:

![picture](/monthly_beer_production/img/preprocess_lag.PNG)

Hasil query:

![picture](/monthly_beer_production/img/preprocess_lag_res.PNG)

Hasil query menghasilkan missing value pada beberapa row pertama karena tidak ada nilai t-1 atau t-6 yang dapat diambil. Oleh karena itu kita perlu menghandle missing value tersebut dengan Spark Missing Value.

Node missing value:

![picture](/monthly_beer_production/img/preprocess_missing.PNG)

Hasil node:

![picture](/monthly_beer_production/img/preprocess_missing_res.PNG)


Kemudian kita lakukan first order difference dengan query berikut:

![picture](/monthly_beer_production/img/preprocess_first_ord.PNG)

Hasil query:

![picture](/monthly_beer_production/img/preprocess_first_ord_res.PNG)


## Modelling

![picture](/monthly_beer_production/img/modelling.PNG)

Kita melakukan dua macam analisis, dengan metanode Predictions  kita membuat model untuk melalukan prediksi dengan menggunakan partisi dari dataset yang ada. Analisis yang dilakukan pada metanode PCA, K-Means, Scatter Plot yang melakukan cluster sesuai Hasil PCA.

### Prediksi dengan metanode Predictions

![picture](/monthly_beer_production/img/modelling1.PNG)

Kita kemudian melakukan partisi training dan test dengan Spark Partition dan membuat model prediksi produksi bir dengan Spark Gradient Boosted Trees Learner. Spark Gradient Boosted Trees Learner dipilih karena menghasilkan error yang paling sedikit dibanding tipe ML yang lain, namun sayangnya KNIME tidak bisa mengubah tipe model tersebut menjadi PMML (Spark Gradient Boosted Trees versi MLlib spark juga tidak bisa dijadikan PMML dengan node Spark MLlib to PMML). Model ditraining dengan partisi training menggunakan MA produksi bir sebelum lag/ MA produksi bir asli sebagai label dan hasil ektraksi tanggal, first order difference, seasonilality removal dan MA produksi bir yang di-lag sebagai fitur. Setelah itu, kita melakukan prediksi dengan Spark Predictor pada partisi test.

Konfigurasi partisi:

![picture](/monthly_beer_production/img/modelling1_partition.PNG)

Konfigurasi model:

![picture](/monthly_beer_production/img/modelling1_model.PNG)

Hasil Prediksi:

![picture](/monthly_beer_production/img/modelling1_model_res.PNG)


### Clustering  dengan metanode PCA,K-Means, Plot


![picture](/monthly_beer_production/img/modelling2.PNG)

Nilai pada Dataframe yang dihasilkan pada preparasi data pertama-tama di normalisasi menjadi nilai antara 0 sampai 1. Kemudian kita melakukan K-Means Clustering terhadap data yang sudah dinormalisasi menjadi 3 cluster. Selain itu, kita juga melakukan Principal Component Analysis (PCA) untuk melakukan dimensionality reduction. Penamaan PC yang dihasilkan dirurutkan berdasarkan eigenvalue yang terbesar ke terkecil, jadi PC0 memiliki eigenvalue yang lebih tinggi dibanding PC1. Hasil dari kedua node tersebut kemudian digabung dengan join, kemudian nilai-nilai yang dinormalisasi dikembalikan ke nilai awalnya dengan melakukan denormalisasi.

Konfigurasi PCA:

![picture](/monthly_beer_production/img/modelling2_PCA.PNG)

Konfigurasi K-Means Clustering

![picture](/monthly_beer_production/img/modelling2_Cluster.PNG)

Hasil join:

![picture](/monthly_beer_production/img/modelling2_join1.PNG)

Hasil denormalisasi

![picture](/monthly_beer_production/img/modelling2_denorm.PNG)

## Evaluasi

### Evaluasi metanode Predictions

![picture](/monthly_beer_production/img/modelling1.PNG)

Model yang dihasilkan dievaluasi dengan Spark Numeric Scorer, selain itu hasil prediksi di-plot di graf untuk memvisualisasikan hasil prediksi dengan ground truth.

Hasil Scoring:

![picture](/monthly_beer_production/img/evaluation1_score.PNG)

Membuat grafik tersebut, kita perlu menggabungkan partisi terlebih dahulu, kemudian mengubah spark dataframe menjadi tabel.

Query untuk menambahkan kolom prediksi ke partisi pertama:

![picture](/monthly_beer_production/img/evaluation1_add_pred.PNG)

Penggabungan partisi dengan Spark Concatenate:

![picture](/monthly_beer_production/img/evaluation1_concate.PNG)

Mengubah dataframe menjadi tabel dengan Spark to Table:

![picture](/monthly_beer_production/img/evaluation1_to_table.PNG)

Grafik yang dihasilkan:

![picture](/monthly_beer_production/img/evaluation1_graph.PNG)

### Evaluasi metanode PCA, K-Means, Scatter Plot

![picture](/monthly_beer_production/img/modelling2.PNG)

Data yang dihasilkan diwarnai berdasarkan dan digambar di suatu grafik dimana x adalah PC0 dan y adalah PC1. Dengan melihat grafik tersebut kita bisa melihat korelasi antara cluster-cluster yang dibentuk. PC0 dan PC1 dipilih karena keduanya adalah PC yang memiliki eigenvalue yang tertinggi sehingga memiliki bobot yang tertinggi untuk mengetahui korelasi antara titik-titik data.

Pewarnaan Cluster:

![picture](/monthly_beer_production/img/evaluation2_colour.PNG)


Grafik yang dihasilkan:

![picture](/monthly_beer_production/img/evaluation2_res.PNG)

Dapat disimpulkan bahwa setiap data point adalah sebuah clusternya sendiri.


## Deployment 

![picture](/monthly_beer_production/img/deployment.PNG)

Hasil prediksi dan cluster yang dibentuk sebelumnya di deploy ke Hive dalam bentuk Paraquet.

Konfigurasi deployment prediksi: 

![picture](/monthly_beer_production/img/deployment1_conf.PNG)

Konfigurasi deployment cluster: 

![picture](/monthly_beer_production/img/deployment2_conf.PNG)

Hasil deployment prediksi: 

![picture](/monthly_beer_production/img/deployment1.PNG)

Hasil deployment cluster: 

![picture](/monthly_beer_production/img/deployment2.PNG)