*repository ini dibentuk untuk penyelesaian tugas Komunikasi Data dan Jaringan Komputer*

# Aplikasi BabyBuddy

<img src="images_res/icon.png" height="100" align="center">

## Anggota Kelompok

| Nama        | NIM         |
| ------------- |:-------------:|
| Yasmin Salamah      | G64170108 |
| Wanda Azizah Yasin      | G64170027      |
| Dwi Fristanti Sagala | G64170047      |

## Sekilas Tentang

Sebuah aplikasi dashboard yang dapat membantu pengurus bayi untuk mencatat waktu tidur, waktu makan, penggantian popok dan dapat memprediksi kebutuhan bayi kedepannya.

![Baby Buddy desktop view](images_res/screenshot.png)

![Baby Buddy mobile views](images_res/screenshot_mobile.png)

Untuk demo aplikasi dapat diakses di [demo of Baby Buddy](http://demo.baby-buddy.net).

Untuk kredensial login adalah :

- Username: `admin`
- Password: `admin`

## Membuat VM ubuntu server

1. Mengunduh VDI ubuntu 18.04 headless dari http://repo.apps.cs.ipb.ac.id/lab/ubuntu-server.vdi.gz.

2. Membuat instance ubuntu baru kemudian melakukan konfigurasi network.

![instance ubuntu](images_res/set1.png)

Buka *network -> advanced* lalu lakukan port forwarding

![port forwarding](images_res/set2.png)

3. Karena komputer host saya menggunakan windows maka saya menggunakan sebuah aplikasi bernama putty untuk mengakses vm linux yang saya buat sebelumnya.

<img src="images_res/putty_icon.png" height="10" align="left">

Untuk mengunduh puTTy dapat dilakukan di [sini](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

4. Saya kemudian mengakses vm dengan mengisi kredensial session

![ssh](images_res/1.png)


## Instalasi

1. Install system packages, untuk sesi instalasi ini kita alias kan python3 menjadi python (opsional).

```
    sudo apt-get install python3 python3-pip nginx uwsgi uwsgi-plugin-python3 git libopenjp2-7-dev
    alias python=python3
```

2. Install pipenv

`   sudo -H pip3 install pipenv`

3. Buat direktori dan clone repository.

```
      sudo mkdir /var/www/babybuddy
      mkdir -p /var/www/babybuddy/data/media
      git clone https://github.com/babybuddy/babybuddy.git /var/www/babybuddy/public
```

opsional : ubah owner direktori setelah membuat direktori

`      sudo chown user:user /var/www/babybuddy`

4. change direktori ke babybuddy/public kemudian set pipenv secara lokal

```
        cd /var/www/babybuddy/public
        export PIPENV_VENV_IN_PROJECT=1
        pipenv install --three
        pipenv shell
```      

5. Edit production setting file dan ubah nilai ``SECRET_KEY`` and ``ALLOWED_HOSTS``

```
        cp babybuddy/settings/production.example.py babybuddy/settings/production.py
        editor babybuddy/settings/production.py
```

Catatan :

Baca dokumentasi Django's documentation untuk setting production terutama mengenai ALLOWED_HOSTS](https://docs.djangoproject.com/en/3.0/ref/settings/#allowed-hosts)

+ ALLOWED_HOSTS

Apabila *ALLOWED_HOSTS[]* :
ketika DEBUG is True and ALLOWED_HOSTS is empty, maka host akan menjadi ``['localhost', '127.0.0.1', '[::1]'].``


+ SECRET_KEY

*SECRET_KEY tidak boleh kosong*
isi dengan value apapun, anggap seperti password.


6. Inisiasi Aplikasi

```
        export DJANGO_SETTINGS_MODULE=babybuddy.settings.production
        python manage.py migrate
        python manage.py createcachetable
```

7. Mengubah permission untuk ownership direktori

```
        sudo chown -R www-data:www-data /var/www/babybuddy/data
        sudo chmod 640 /var/www/babybuddy/data/db.sqlite3
        sudo chmod 750 /var/www/babybuddy/data
```
8. Membuat dan mengkonfigurasi uWSGI

`      sudo editor /etc/uwsgi/apps-available/babybuddy.ini`

    Contoh Konfigurasi, jangan lupa untuk mengisi direktori:

```
        [uwsgi]
        plugins = python3
        project = babybuddy
        base_dir = /var/www/babybuddy

        chdir = %(base_dir)/public
        virtualenv = %(chdir)/.venv
        module =  %(project).wsgi:application
        env = DJANGO_SETTINGS_MODULE=%(project).settings.production
        master = True
        vacuum = True
```


9. Konfigurasi Symlink dan restart uWSGI

```
        sudo ln -s /etc/uwsgi/apps-available/babybuddy.ini /etc/uwsgi/apps-enabled/babybuddy.ini
        sudo service uwsgi restart
```

10. Membuat dan mengkonfigurasi NGINX

`sudo editor /etc/nginx/sites-available/babybuddy`

    Contoh konfigurasi:

```javascript
        upstream babybuddy {
            server unix:///var/run/uwsgi/app/babybuddy/socket;
        }

        server {
            listen 80;
            server_name localhost;

            location / {
                uwsgi_pass babybuddy;
                include uwsgi_params;
            }

            location /media {
                alias /var/www/babybuddy/data/media;
            }
        }
```

11. Konfigurasi Symlink dan Restart NGINX

```
        sudo ln -s /etc/nginx/sites-available/babybuddy /etc/nginx/sites-enabled/babybuddy
        sudo service nginx restart
```

12. Buka localhost.


## Konfigurasi

Spesifikasi instance VPS :
- Minimal Ubuntu 18.04
- Minimal Storage 512MB

Spesifikasi Technology :
- Python 3.6+
- nginx
- uwsgi
- sqlite

## API

BabyBuddy menggunakan DJANGO REST FRAMEWORK untuk REST API
Dokumentasi lebih lanjut dapat diakses di [sini](https://www.django-rest-framework.org/)

#### Authentication

Untuk menggunakan key untuk API Request, dev hanya perlu meng set request `Authorization`
header menjadi `Token <user-key>`. Seperti contoh :

`Authorization: Token 2h23807gd72h7hop382p98hd823dw3g665g56`

Apabila `Authorization` tidak valid, maka API akan mengembalikkan `403 Forbidden` pada response body.

#### `GET`,`OPTIONS` dan `POST` requests :

- `/api/children/`
- `/api/changes/` (Diaper Changes)
- `/api/feedings/`
- `/api/notes/`
- `/api/sleep/`
- `/api/temperature/`
- `/api/timers/`
- `/api/tummy-times/`
- `/api/weight/`


## Cara Pemakaian

- Tampilan aplikasi web

![Tampilan Aplikasi Web](images_res/lamanutama.png)

- Fungsi-fungsi utama

Fungsi utama aplikasi dibagi menjadi 3 :

1. Mengisi entry tentang perkembangan anak
2. Mengisi entry aktivitas
3. Timer

+ Entry Perkembangan Anak

Berikut fungsi-fungsi yang ada pada entry anak :

![entryperkembangananak(images_res/lamanutama.png)

Berikut penjabaran fungsi :

| Fungsi    | Deksripsi         | Gambar          |
| ------------- |:-------------:| --------------- |
| Menambahkan Anak | Menambahkan objek anak dan mengisi nama | ![1](images_res/children2) |
| Menambahkan Note | Kita dapat mengisi note atas apapun yang dilakukan anak kita hari ini atau mengenai anak kita | ![2](images_res/children3) |
| Menambahkan temperature | Menambahkan suhu tubuh beserta tanggal pengambilan suhu | ![3](images_res/children4) |
| Menambahkan Berat badan | Menambahkan berat badan beserta tanggal pengambilan  | ![4](images_res/children5) |

+ Entry aktivitas

Berikut fungsi-fungsi yang ada pada entry anak :

![entryperkembangananak(images_res/act1.png)

Berikut penjabaran fungsi :

| Fungsi    | Deksripsi         | Gambar          |
| ------------- |:-------------:| --------------- |
| Ganti Popok | Menambahkan kapan kita mengganti popok dan waktunya | ![1](images_res/act2) |
| Memberi Makan | Menambahkan metode pemberian makan,detail dan kapan pemberian makan | ![2](images_res/act4) |
| Tidur | Menambahkan waktu tidur dan durasi | ![3](images_res/act5) |
| Tummy Time | Saya ga ngerti ini apa | ![4](images_res/act6) |


+ Timer

| Fungsi    | Deksripsi         | Gambar          |
| ------------- |:-------------:| --------------- |
| Timer | Timer | <img src="images_res/timer1.png" align="center"> |


+ Hasil dashboard

setelah sekian entry data yang kita masukkan maka hasilnya akan ditampilkan pada dashboard sebagai berikut ini :

![](images_res/dashboard_end.png)


## Pembahasan

#### Menurut kami BabyBuddy selama proses deployment mempunyai :

- kelebihan

+ config files lengkap dan terstruktur dengan baik sesuai dengan dokumentasi tech masing-masing yang digunakan. Jadi memudahkan untuk deployment di platform manapun.
+ `docker-compose` tersedia memudahkan pengguna yang ingin melakukan deployment di docker. Hanya tinggal set value di production setting file lalu initialisasi app.
+ Dokumentasi spesifikasi nilai parameter yang digunakan untuk API lengkap.

- kekurangan


#### Perbandingan dengan aplikasi sejenis

Hampir semua personal dashboard application sejenis hampir mirip secara struktur.

Namun aplikasi serupa yang mirip dengan babybuddy :
+ BI for Baby [bi for baby](https://timoelliott.com/blog/2009/11/bi-for-baby-dashboards-to-track-your-childs-kpis.html)


## Referensi

+ [github babybuddy](https://github.com/babybuddy/babybuddy#authentication)
+ [uWSGI documentation](http://uwsgi-docs.readthedocs.io/en/latest/)
+ [nginx documentation](https://nginx.org/en/docs/)
+ [nginx documentation](https://nginx.org/en/docs/)
