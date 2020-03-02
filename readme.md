*repository ini dibentuk untuk penyelesaian tugas Komunikasi Data dan Jaringan Komputer*

# Aplikasi BabyBuddy

## Anggota Kelompok

| Nama        | NIM         |
| ------------- |:-------------:|
| Yasmin Salamah      | G64170108 |
| Wanda Azizah Yasin      | G64170      |
| zebra stripes | are neat      |

## Sekilas Tentang

<img src="icon.png" height="150" align="left">

Sebuah aplikasi dashboard yang dapat membantu pengurus bayi untuk mencatat waktu tidur, waktu makan, penggantian popok dan dapat memprediksi kebutuhan bayi kedepannya.

![Baby Buddy desktop view](screenshot.png)

![Baby Buddy mobile views](screenshot_mobile.png)

Untuk demo aplikasi dapat diakses di [demo of Baby Buddy](http://demo.baby-buddy.net).

Untuk kredensial login adalah :

- Username: `admin`
- Password: `admin`


## Membuat VM ubuntu server

1. Mengunduh VDI ubuntu 18.04 headless dari http://repo.apps.cs.ipb.ac.id/lab/ubuntu-server.vdi.gz.

2. Membuat instance ubuntu baru kemudian melakukan konfigurasi network.

![instance ubuntu](set1.png)

Buka *network -> advanced* lalu lakukan port forwarding

![port forwarding](set2.png)

3. Karena komputer host saya menggunakan windows maka saya menggunakan sebuah aplikasi bernama putty untuk mengakses vm linux yang saya buat sebelumnya.

<img src="putty_icon.png" height="50" align="left">

Untuk mengunduh puTTy dapat dilakukan di [sini](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

4. Saya kemudian mengakses vm dengan mengisi konfigurasi ssh

![ssh](1.png)


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
            server_name babybuddy.example.com;

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


##  Maintenance (opsional)

Setting tambahan untuk maintenance secara periodik, misalnya:
- buat backup database tiap pekan
- hapus direktori sampah tiap hari
- dll


## Otomatisasi (opsional)

Skrip shell untuk otomatisasi instalasi, konfigurasi, dan maintenance.


## Cara Pemakaian

- Tampilan aplikasi web

![Tampilan Aplikasi Web](lamanutama.png)

- Fungsi-fungsi utama
- Isi dengan data real/dummy (jangan kosongan) dan sertakan beberapa screenshot


## Pembahasan

- Pendapat anda tentang aplikasi web ini
    - kelebihan
    - kekurangan
- Bandingkan dengan aplikasi web lain yang sejenis


## Referensi

+ [github babybuddy](https://github.com/babybuddy/babybuddy#authentication)
+ [uWSGI documentation](http://uwsgi-docs.readthedocs.io/en/latest/)
+ [nginx documentation](https://nginx.org/en/docs/)
+ [nginx documentation](https://nginx.org/en/docs/)
