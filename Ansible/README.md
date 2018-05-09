# Ansible

### Anggota :

##### 1. Reinhart Caesar - 05111540000132   
##### 2. Joshua Resamuel - 05111540000172
# 1. Membuat 3 Virtual Machine
> 2 Virtual Machine Ubuntu 14.04 sebagai Worker dan 1 Virtual Machine Debian 9 sebagai Database Server. Aplikasi virtualisasi yang digunakan adalah **VirtualBox**.

# 2. Konfigurasi Virtual Machine
> Install SSH pada VM :
```
   sudo apt-get install openssh-server openssh-client
```

# 3. Ansible Inventory

1. Menginstall **ansible** dan **sshpass**
   ```
      sudo apt install ansible
      sudo apt-get install sshpass
   ```
2. Membuat folder **simple-playbook**.

    ```
    mkdir simple-playbook
    cd simple-playbook/
    ```
3. Membuat file **hosts** sebagai **Ansible Inventory**.
    ```
    nano hosts
    ```
    isi:

    ```
    worker1 ansible_host=192.168.20.129 ansible_ssh_user=vm ansible_become_pass=vm
    worker2 ansible_host=192.168.20.130 ansible_ssh_user=vm ansible_become_pass=vm    
  
4. Validasi dengan melakukan ping.

    ```
    ansible -i ./hosts -m ping all -k
    ```
    Keterangan:
    * **-i** : declare Ansible Inventory.
    * **-m** : declare module command (command **ping**).
    * **all** : penanda ansible dijalankan di host mana. Parameter **all** dapat diganti dengan nama host.
    * **-k** : menanyakan password login ssh.

# 4. Grouping Host

* Membuka file ```hosts```
```
    nano hosts
```
* Tambahkan group dalam tanda **[ ]**. Contoh : group **[worker]**.

```
[worker]
worker1 ansible_host=192.168.20.129 ansible_ssh_user=vm ansible_become_pass=vm
worker2 ansible_host=192.168.20.130 ansible_ssh_user=vm ansible_become_pass=vm
```

## 5. Install Laravel 5.6

Software yang digunakan untuk menjalankan Laravel 5.6:

    * Nginx
    * PHP 7.2
    * Composer
    * Git

Software akan diinstall pada hosts **worker** menggunakan file **laravel.yml**.

1. Membuat playbook **laravel.yml**.

    ```
    nano laravel.yml
    ```
2. Isi script:

    ```yml
    ---
    - hosts: worker
      tasks:
        
        - name: Instalasi Nginx, Git, Zip, dan Unzip
          become: true
          apt:
            name: "{{ item }}"
            state: latest
            update_cache: true
          with_items:
            - nginx
            - git
            - python-software-properties
            - software-properties-common
            - zip
            - unzip
          notify:
            - Restart nginx

        # Install PHP 7.2
        - name: Menambah Repositori PHP 7
          become: true
          apt_repository:
            repo: 'ppa:ondrej/php'
            update_cache: true

        - name: Install Package PHP 7.2
          become: yes
          apt:
            name: "{{ item }}"
            state: latest
          with_items:
            - php7.2
            - php-pear
            - php7.2-curl
            - php7.2-dev
            - php7.2-gd
            - php7.2-mbstring
            - php7.2-zip
            - php7.2-mysql
            - php7.2-xml
            - php7.2-intl
            - php7.2-json
            - php7.2-cli
            - php7.2-common
            - php7.2-fpm
          notify:
            - Restart PHP-fpm

    handlers:
        - name: Restart nginx
          become: true
          service:
            name: nginx
            state: restarted

        - name: Restart PHP-fpm
          become: true
          service:
            name: php7.2-fpm
            state: restarted
    ```
    Keterangan:
    
    * **Zip** dan **unzip** digunakan untuk mempercepat proses instalasi Composer.
    * ```Handlers``` digunakan untuk mendefinisikan task yang dipanggil di modul ```notify```.

## 6. Git Clone - Aplikasi Laravel

* Membuka playbook```laravel.yml```
* Menambahkan isi Script:

```yml
    # Git Clone
    - name: Membuat direktori
      become: true
      file:
        path: "/var/www/laravel"
        state: directory
        owner: "{{ ansible_ssh_user }}"
        group: "{{ ansible_ssh_user }}"
        recurse: yes

    - name: Git Clone
      git:
        dest: "/var/www/laravel"
        repo: https://github.com/udinIMM/Hackathon.git
        force: yes
```


## 7. Instalasi Composer dan Konfigurasi Laravel Environment

1. Membuka playbook```laravel.yml``` dan menambahkan isi Script:

    ```yml
        - name: Download Composer
          script: script/composer.sh

        - name: Setting Composer menjadi Global
          become: true
          command: mv composer.phar /usr/local/bin/composer

        - name: Set Composer Permission
          become: true
          file:
            path: /usr/local/bin/composer
            mode: "a+x"

        - name: Install Laravel Dependencies
          composer:
            working_dir: "/var/www/laravel"
            no_dev: no
        
        # Konfigurasi Environment
        - name: konfigurasi .env
          command: cp "/var/www/laravel/.env.example" "/var/www/laravel/.env"
        
        - name: php artisan key generate
          command: php "/var/www/laravel/artisan" key:generate

        - name: php artisan clear cache
          command: php "/var/www/laravel/artisan" cache:clear
        
        - name: set APP_DEBUG=false
          lineinfile:
            dest: "/var/www/laravel/.env"
            regexp: '^APP_DEBUG='
            line: APP_DEBUG=false

        - name: set APP_ENV=production
          lineinfile:
            dest: "/var/www/laravel/.env"
            regexp: '^APP_ENV='
            line: APP_ENV=production

        - name: set Database Connection
          lineinfile:
            dest: "{{ laravel_root_dir }}/.env"
            regexp: '^DB_HOST='
            line: DB_HOST=ipku

        - name: set Database Name
          lineinfile:
            dest: "{{ laravel_root_dir }}/.env"
            regexp: '^DB_DATABASE='
            line: DB_DATABASE=hackathon

        - name: set Database Username
          lineinfile:
            dest: "{{ laravel_root_dir }}/.env"
            regexp: '^DB_USERNAME='
            line: DB_USERNAME=regal

        - name: set Database Password
          lineinfile:
            dest: "{{ laravel_root_dir }}/.env"
            regexp: '^DB_PASSWORD='
            line: DB_PASSWORD=bolaubi

        - name: Mengganti Permission bootstrap/cache directory
          file:
            path: "/var/www/laravel/bootstrap/cache"
            state: directory
            mode: "a+x"

        - name: Mengganti Permission vendor directory
          command: chmod -R 777 "/var/www/laravel/vendor"

        - name: Mengganti Permission storage directory
          command: chmod -R 777 "/var/www/laravel/storage"
    ```
    
2. Membuat file ```composer.sh``` dalam folder ```script```.

    ```bash
    mkdir script
    cd script/
    nano composer.sh
    ```
* isi script:
    
    ```
    #!/bin/sh

    EXPECTED_SIGNATURE=$(wget -q -O - https://composer.github.io/installer.sig)
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    ACTUAL_SIGNATURE=$(php -r "echo hash_file('SHA384', 'composer-setup.php');")

    if [ "$EXPECTED_SIGNATURE" != "$ACTUAL_SIGNATURE" ]
    then
        >&2 echo 'ERROR: Invalid installer signature'
        rm composer-setup.php
        exit 1
    fi

    php composer-setup.php --quiet
    RESULT=$?
    rm composer-setup.php
    exit $RESULT
    ```

## 8. Konfigurasi Nginx

1. Membuat file konfigurasi ```nginx.conf``` dalam folder ```nginx```.

    ```
    mkdir nginx
    cd nginx/
    nano nginx.conf
    ```
    * isi script:

    ```
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        
        root /var/www/laravel/public;

        index index.php;

        server_name {{ inventory_hostname }};

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        error_log /var/log/nginx/{{ inventory_hostname }}_error.log;
        access_log /var/log/nginx/{{ inventory_hostname }}_access.log;
    }
    ```

2. Membuka playbook```laravel.yml``` dan mengisikan script:

    ```yml
        - name: Konfigurasi Nginx
          become: true
          template:
            src: nginx/nginx.conf
            dest: /etc/nginx/sites-enabled/default
          notify:
            - Restart nginx
            - Restart PHP-fpm
    ```
