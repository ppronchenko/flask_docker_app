# Установка

Как установить - по инструкции отсюда: https://cloud.google.com/compute/docs/quickstart-linux

Внимание! В инструкции установка Debian, а нам нужна Ubuntu 18.04.

Далее в консоли, которая открывается прямо в браузере, выполнить:

<pre>
sudo apt-get update && sudo apt-get -y upgrade
</pre>

Эта команда обновит пакетный менеджер apt-get. После этотго установить пакет pip:

<pre>
sudo apt install python-pip
</pre>

pip - это менеджер пакетов python.  С его помощью можно будет устанавливать python библиотеки.

Теперь файл нужно перенести в виртуалку Google Cloud. Для этого устанавливаем сконируем полезный репозиторий

<pre>
git clone https://github.com/chentinghao/download_google_drive.git
<pre>

Запускаем скачивание файла - zip архива с данными.
<pre>
python download_google_drive/download_gdrive.py 1YiclJxLxWZcHX8ObNiQaS0ZOVyyD7TX4 data.zip
</pre>

И распаковываем архив

<pre>
sudo apt-get install unzip; unzip data.zip -d  /tmp/data
</pre>

Установим docker и docker-compose

<pre>
sudo apt-get install docker docker-compose
</pre>

Теперь скачивает репозиторий курса Нетологии

<pre>
dju0204@instance-ubuntu:~$ git clone https://github.com/Dju999/flask_docker_app.git
Cloning into 'flask_docker_app'...
remote: Counting objects: 81, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 81 (delta 0), reused 0 (delta 0), pack-reused 77
Unpacking objects: 100% (81/81), done.
dju0204@instance-ubuntu:~$ cd flask_docker_app/postgres_interactions
</pre>

Переходим в директорию занятия
<pre>
cd flask_docker_app/postgres_interactions
</pre>

Запускаем построение контейнера
<pre>
sudo docker-compose --project-name postgres-client -f docker-compose.yml up --build -d
</pre>

В консоли побежит информация о сборке контейнера. Для теста запустим контейнер и подключимся к командной строке Alpine-Linux:
<pre>
sudo docker-compose --project-name postgres-client -f docker-compose.yml run --rm postgres-client
Starting postgresclient_postgres_host_1 ... done
/ # echo "Hello, Netology!"
Hello, Netology!
</pre>

Проверим, что директория с данными Kaggle подключилась:
<pre>
/ # ls /data
credits.csv          links.csv            movies_metadata.csv  ratings_small.csv
keywords.csv         links_small.csv      ratings.csv
</pre>

Как видно csv-файлы присутствуют, контейнер запущен! можно начинать работу.
