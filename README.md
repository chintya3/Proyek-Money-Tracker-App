# Proyek-Money-Tracker-App
### Buat Project Baru

Buat project baru sesuai dengan ketentuan submission, dan gunakan project tersebut

### Buat Bucket

1. Navigation Menu → Cloud Storage → Buckets → Create Buckets
2. Beri nama sesuai ID project agar mudah → continue
3. Pilih region → asia-southeast2 (Jakarta) → continue
4. Storage class biarkan saja → continue
5. Hapus centang pada “Enforce public access prevention on this bucket” → pilih fine-grained → continue    
6. Protection tools → object versioning → isikan seperti gambar di bawah → continue
7. Create

### Masuk ke dalam bucket yang sudah dibuat

Masuk ke menu **PERMISSIONS**

1. Grant Access
2. New Principals → isi dengan allUsers
3. Role → Storage Object Viewer
4. Save → Allow Public Access

### Buat VM Instance

1. Navigation Menu → Compute Engine → VM Instance
2. Berikan nama Instance sesuka hati anda, ikuti settingan seperti di bawah ini
3. Firewall → centang Allow HTTP traffic dan Allow HTTPS traffic
4. Create

### Buat instance SQL

1. Navigation Menu → SQL
2. Create Instance
3. Choose MySQL
4. Tentukan Instance ID dan password sesuka hati, yang penting ingat, dan ikuti settingan seperti di bawah ini
5. Region → asia-southeast2 (Jakarta)
6. Data Protection → hapus centang **Enable deletion protection,** agar nanti setelah selesai submission bisa langsung dihapus
7. **Create Instance**
8. Setelah selesai, masuk ke Databases
9. Create Database → beri nama sesuka hati
10. **Create**
11. Connections → Add A Network → Beri nama sesuka hati
12. Isikan network dengan External IP VM → DONE
13. **SAVE** → tunggu hingga selesai
14. Buka Cloud Shell
15. Jalankan perintah
    
    gcloud sql connect <nama-sql-instance> --user=root
    
16. Jika ada prompt, langsung Y aja
17. Masukkan password yang tadi dibuat, tenang jika tidak ada tulisan atau sensor itu bukan masalah yang penting password diinputkan dengan benar
18. Nanti akan tampil seperti ini
19. Masukkan perintah
    
    USE <nama-database>;
    
    contoh USE iqbal;
    
    *pastikan terdapat pesan “Database changed”
    
20. Masukkan query ini
    
    ```sql
    CREATE TABLE records (
        id INT AUTO_INCREMENT PRIMARY KEY NOT NULL,
        name VARCHAR(25) NOT NULL,
        amount DOUBLE NOT NULL,
        date DATETIME NOT NULL,
        notes TEXT,
        attachment VARCHAR(255)
    );
    ```
    
    akan muncul pesan seperti ini “Query OK, 0 rows affected”
    
21. Ketik exit

### Buat Firewall

1. Navigation Menu → VPC Network → Firewall
2. Create Firewall Rule
3. Berikan nama sesuka hati, dan ikuti settingan di bawah ini
4. Create

### Konfigurasi IAM

Navigation Menu → IAM & Admin

Service Accounts

1. Service Accounts → Create Service Accounts
2. Beri nama sesuka hati → Create and Continue
3. Role → Storage Object Admin → Continue
4. Done
5. Setelah terbuat → ke titik 3 paling kanan (pastikan mengarah ke service account yang baru saja dibuat → manage keys
6. Add Key → Create new key
7. JSON → Create
8. Simpan file JSON di tempat yang mudah ditemukan, nanti akan digunakan pada tahap selanjutnya

Custom Roles 1

1. Roles → Create Role 
2. Beri nama dan ID sesuka hati
3. Tambahkan 19 permissions ini
4. Setelah semua permissions dicentang → Add
5. Create

Custom Roles 2

1. Roles → Create Roles
2. Beri nama dan ID sesuka hati
3. Tambahkan 6 permission ini
4. Add → Create

Menerapkan Previlige Untuk Reviewer Dicoding

1. Untuk reviewer Dicoding
    a. IAM → Grant Access
    b. Role 1 dan 2 merupakan custom role yang telah dibuat sebelumnya
    c. Save

### Clone Repository Back-End

*disini saya melakukan config melalui local

1. Buka VSCode
2. Buka terminal → jalankan perintah ini
    
    git clone -b money-tracker-api https://github.com/dicodingacademy/a133-gcp-labs.git
    
Masuk ke dalam folder a133-gcp-labs

1. Copy dan paste isi semua file .json yang sebelumnya sudah dibuat ke dalam **serviceaccountkey.json**
2. modules → **imgUpload.js** → pada line 11 masukkan id project anda, line 16 masukkan nama bucket anda
3. routes → **record.js** → line 14 hingga 17 masukkan setting cloud sql
4. Jika sudah selesai, buat menjadi .zip folder a133-gcp-labs

### Setting VM Instance sebagai Back-End

1. Navigation Menu → Compute Engine → VM Instance
2. klik SSH pada VM yang sudah dibuat
3. Upload file .zip yang sudah dibuat
    
    *jika ada pesan eror, klik retry dan upload ulang
4. Selagi mengunggu upload selesai, jalankan perintah di bawah ini satu per satu

```sudo apt-get install unzip```
tenang proses ini memang lama
​
```unzip a133-gcp-labs.zip```
pastikan proses uploads selesai
​
```cd a133-gcp-labs```
​
```curl -s https://deb.nodesource.com/setup_16.x | sudo bash```
ini juga agak lama
​
```sudo apt install nodejs -y```
​
```npm install```
​
```sudo npm install pm2 -g```
​
```pm2 start app.js``
​
5. Tutup SSH, buka url dibawah dengan tab baru
   http://<external ip>:8000/
6. Harusnya menampilkan “Response Success!”
7. Coba tambahkan /dashboard di belakang
   http://<external ip>:8000/dashboard
8. Harusnya menampilkan "[{"month_records":0,"total_amount":null}]”
9. Pastikan menampilkan sesuai dengan poin 6 dan 8


### Setting App Engine sebagai Front-End

1. Buka Cloud Shell → jalankan perintah ini
    
    git clone -b money-tracker https://github.com/dicodingacademy/a133-gcp-labs.git
    
    git clone -b money-tracker https://github.com/dicodingacademy/a133-gcp-labs/tree/money-tracker
    
    cd a133-gcp-labs
    
2. Masuk ke Open Editor
3. Folder a133-gcp-labs → app.yaml
4. Ganti dengan code ini
    
    ```yaml
    runtime: php74
    service: default
    
    handlers:
      - url: /assets
        static_dir: assets
    ```
    
5. Buka application → config → config.php
6. Pada line 27 ganti dengan kode di bawah ini
6. Pada line 27 ganti dengan kode di bawah ini
    
    ```php
    $config['base_url'] = ((isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] == "on") ? "https" : "http");
    $config['base_url'] .= "://" . $_SERVER['HTTP_HOST'];
    $config['base_url'] .= str_replace(basename($_SERVER['SCRIPT_NAME']), "", $_SERVER['SCRIPT_NAME']);
    ```
    
7. models → Record_model.php
8. Pada line 11 masukkan http://<external ip>:8000/
9. Pastikan sudah disave semua
10. Klik Open Terminal → jalankan
    
    gcloud app deploy → pilih asia-southeast2 (seharusnya no 8) → Y
    
    gcloud app browse → klik link yang diberikan
    
11. Testing dengan menambahkan data
12. Pada link attachment, pastikan dapat diakses dan menampilkan gambar yang diupload
13. Done
