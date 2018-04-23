## Penyelesaian Tugas Docker

### No. 1
- Step 1 - Membuat direktori baru untuk Tugas Docker
```
    mkdir tugas-docker
```
- Step 2 - Membuat File **Dockerfile**
```
    nano Dockerfile
```
- Step 3 - Konfigurasi**Dockerfile**
```
    FROM ubuntu:16.04

    RUN apt-get update -y
    RUN apt-get -y upgrade

    RUN apt-get install -y wget zip
    # Instalasi Python
    RUN apt-get install -y python python-pip python-dev build-essential
    # Instalasi MySQL
    RUN apt-get install -y libmysqlclient-dev
    RUN pip install --upgrade pip

    # Direktori Reservasi
    COPY www/reservasi reservasi
    WORKDIR reservasi

    # Install Dependencies untuk Python Flask
    RUN pip install -r req.txt

    # Menjalankan Python saat docker dijalankan
    ENTRYPOINT [ "python" ]

    # Menjalankan server.py
    CMD [ "server.py" ]

    # Port 80
    EXPOSE 80
```
- Step 4 - Mengunduh Web File **Reservasi** dari [sini](https://cloud.fathoniadi.my.id/reservasi.zip) dan menaruhnya di folder www
- Step 5 - Buat image pada docker
```
    docker build -t tugas-docker ./
```

##### No. 2 s.d. No. 4
- Step 1 - Membuat File **docker-compose.yml**
```
    nano docker-compose.yml
```
- Step 2 - Konfigurasi File **docker-compose.yml**
```
    version: '3.0'
services:
    db:
        image: mysql:5.7
        restart: always
        volumes:
            - ./reservasi:/docker-entrypoint-initdb.d
            - dbdata:/var/lib/mysql
        environment:
            MYSQL_ROOT_PASSWORD: buayakecil
            MYSQL_DATABASE: reservasi
            MYSQL_USER: userawan
            MYSQL_PASSWORD: buayakecil

    worker:
        image: tugas-docker
        depends_on:
            - db
        environment:
            DB_HOST: db
            DB_USERNAME: userawan
            DB_PASSWORD: buayakecil
            DB_NAME: reservasi

    load-balancer:
        image: nginx:stable-alpine
        ports:
            - 8008:80
        depends_on:
            - worker
        volumes:
            - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro

volumes:
    dbdata:

```
- Step 3 - Menjalankan File **docker-compose.yml**
```
    docker-compose up -d
    
```

##### Catatan
- Membuat Scale untuk Worker, **worker=3** untuk membuat 3 worker
```
    docker-compose scale worker=3
```
- Restart nginx pada container
```
    docker-compose restart nginx
```
- Mengecek Worker, apakah sudah berjalan atau tidak
```
    docker ps
```

##### Catatan No. 3
1. Load Balancer - Image Container dari Docker Hub -> **nginx:stable-alpine**
2. Konfigurasi **nginx.conf** akan disimpan pada **/etc/nginx/conf.d/default.conf:ro** di container
3. Konfigurasi untuk **nginx.conf**
```
    server {
        listen  80 default_server;
        location / {
            proxy_pass http://worker:80;
        }
    }
```

##### Catatan No. 4
1. db - Image Container dari Docker Hub -> **mysql:5.7**
2. db - Volume untuk service db, mengunakan konfigurasi **./reservasi:/docker-entrypoint-initdb.d**, untuk mengkonfigurasi Database untuk sistem Reservasi, dengan membaca dan meng-import seluruh isi dari direktori yang memiliki format .sql dan menyimpannya pada **dbdata** yang ada pada direktori **/var/lib/mysql**.
