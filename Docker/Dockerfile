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