# P2M3 - Steps

## Flowchart

![plot](flowchart.png)

## Steps
Step by step pengerjaan Milestone 3 : 

1. Buat sebuah folder baru di local computer, misalkan **project-m3**.

2. Pastikan Anda sudah melakukan clone repository https://github.com/ardhiraka/DEBlitz.

3. Pada path `DEBlitz/compose_file/airflow/`, copy 2 buah file yaitu `.env` dan `airflow_ES.yaml` ke dalam folder baru tadi yang berada di local computer (folder `project-m3`) sehingga struktur folder `project-m3` akan menjadi
   ```
   project-m3
   |
   ├── .env
   ├── airflow_ES.yaml
   ```

4. Ubah sintaks pada docker compose `airflow_ES.yaml` agar data yang dimasukkan ke dalam PostgreSQL dapat dibuka dilain waktu.
  
   *BEFORE*
   ```
   services: 
     postgres:
       image: postgres:13
       container_name: postgres
       ports:
         - "5434:5432"
   ```

   *AFTER*
   ```
   services: 
     postgres:
       image: postgres:13
       container_name: postgres
       volumes:
         - ./postgres_project_m3:/var/lib/postgresql/data
       ports:
         - "5434:5432"
   ```
5. Ubah versi Elasticsearch dan juga Kibana ke versi `7.13.0`. Anda dapat mengecek caranya pada poin `B.3` pada [link](https://github.com/danupurnomo/lecture-notes/blob/main/elastic-kibana-notes.md).

6. Perhatikan bagian `x-common` pada Docker Compose.
   ```
   x-common:
     &common
     image: apache/airflow:2.3.4
     user: "${AIRFLOW_UID:-50000}:0"
     env_file: 
       - .env
     volumes:
       - ./dags:/opt/airflow/dags
       - ./logs:/opt/airflow/logs
       - ./plugins:/opt/airflow/plugins
       - /var/run/docker.sock:/var/run/docker.sock
   ```

   Terlihat bahwa docker akan melakukan mapping pada folder `dags`, `logs`, dan `plugins`. Oleh karena itu, buat terlebih dahulu 3 folder ini pada folder `project-m3` sehigga struktur foldernya akan menjadi : 

   ```
   project-m3
   |
   ├── .env
   ├── airflow_ES.yaml
   ├── /dags
   ├── /logs
   ├── /plugins
   ```

7. Open Docker Desktop.

8. Buka Terminal atau Commad Prompt. Jalankan docker compose `airflow_ES.yaml` yang terletak di folder `project-m3`.

9. Masukkan `raw_data.csv` ke dalam PostgreSQL yang berasal dari Docker. 
    - Secara default, PostgreSQL yang dijalankan via Docker ini sudah terdapat beberapa tabel seperti gambar dibawah ini. ![plot](Default%20PostgreSQL%20Tables.png).
    - Buat table baru yaitu `table_m3` untuk menampung `raw_data.csv`. Silakan buat sintaks DDL & DML-nya.

10. Untuk mengakses suatu service baik dari sisi user (developer) maupun dari sisi service lain, silakan buka `airflow_ES.yaml`. Perhatikan bagian `services`
    * Contoh dibawah ini adalah bagian `postgres`
      ```
      postgres:
        image: postgres:13
        container_name: postgres
        ports:
          - "5434:5432"
        healthcheck:
          test: ["CMD", "pg_isready", "-U", "airflow"]
          interval: 5s
          retries: 5
        env_file:
          - .env
      ```

    * Hostname
      - Jika user (developer) ingin mengakses service `postgres` ini, maka masukkan `localhost` sebagai hostname-nya.
      - Jika suatu service (misalka `Apache Airflow`) ingin mengakses service `postgres` ini, masukkan nama servicenya yaitu `postgres`.

    * Port
      - Tertulis 
        ```
        ports:
          - "5434:5432"
        ```
      - Angka sebelum titik dua (`:`) yaitu `5434` adalah port yang bisa dipakai oleh seorang user (developer).
      - Angka setelah titik dua (`:`) yaitu `5432` adalah port yang bisa dipakai oleh suatu service (Apache Airflow).

    * Lakukan konfigurasi sesuai dengan panduan `Hostname` dan `Port` diatas untuk service `Apache Airflow`, `Elasticsearch`, dan `PostgreSQL`.
