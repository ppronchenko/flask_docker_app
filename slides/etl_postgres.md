# ETL процедуры

ETL (Extract, Transform, Load) - общее название для процессов загрузки сырых данных в БД, а так же выгрузки результатов для дальнейшего использования.

## Загрузка данных.

Загружать данные в Postgres можно из CSV файлов, ниже пример который загружает данные из csv:

На первом этапе создаём табличку с данными

<pre>
adzhumurat@adzhumurat-HP-G5: psql -c '
  CREATE TABLE IF NOT EXISTS ratings (
    userId bigint,
    movieId bigint,
    rating float(25),
    timestamp bigint
  );'
</pre>

И на втором этапе заливаем CSV в созданную таблицу:

<pre>
adzhumurat@adzhumurat-HP-G5: psql -c "\\copy ratings FROM '/data/ratings.csv' DELIMITER ',' CSV HEADER"
</pre>

Примечание: перед тем, как загружать данные, нужно очень внимательно изучить csv файл - чтобы точно узнать типы данных и их размерности

## Выгрузка данных

Выгрузку данных можно производить с помощью команды copy

<pre>
\copy (select * from ratings limit 100) copy to 'ratings_file' with header delimiter as ','
</pre>

ETL процессы позволяют использовать Postgres (или другие БД) как средство вычисления: получаем данные, обрабатываем внутри Postgres используя мощный движок вычислений и выгружаем результат для дальнейшего использования - например, в алгоритмах машинного обучения.
