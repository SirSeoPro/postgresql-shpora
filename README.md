# PostgreSQL - Шпаргалка 

Подключение к СУБД, посмотреть базы:

```
sudo -u postgres psql
\l
```

Запуск восстановления БД из дампа:

```
# Копируем дамп в общую папку
sudo mkdir -p /var/tmp/pg_restore
sudo cp /home/ikoshcheev/wms-upp_2025_12_29_163001.dump /var/tmp/pg_restore/
sudo chown postgres:postgres /var/tmp/pg_restore/wms-upp_2025_12_29_163001.dump
sudo chmod 640 /var/tmp/pg_restore/wms-upp_2025_12_29_163001.dump

# Теперь восстанавливаем уже из этого места + логирование
sudo -u postgres pg_restore \
  -d wms-test \
  --clean --if-exists \
  --no-owner --no-privileges \
  -v \
  /var/tmp/pg_restore/wms-upp_2025_12_29_163001.dump \
  > /home/ikoshcheev/restore_log_20251229.txt 2>&1
```

Снятие дампа:

```
# Делаем дамп базы wms-test
sudo -u postgres pg_dump \
  -Fc \                  # формат custom (самый удобный)
  -Z 6 \                 # степень сжатия (6 — хороший баланс скорость/размер)
  -v \                   # verbose — показывает, что происходит
  -f /home/ikoshcheev/backups/wms-test_2025-12-29.dump \
  wms-test
```

Пример скрипта для автоматизации бекапов (пайтон):

```
#!/bin/python3

import psycopg2
from psycopg2 import Error
import datetime
import os

datenow = datetime.datetime.now().strftime("%Y_%m_%d_%H%M%S")

os.system('find /mnt/BackUP/ -type f -mtime +2 -delete')

#### Connect to DB
try:
    # Подключиться к существующей базе данных
        connection = psycopg2.connect(user="wms-backup", password="СЛОЖНЫЙПАРОЛЬ", host="ИМЯСЕРВЕРА", port="5432", database="postgres")
        cursor = connection.cursor()
        getALLDB = "SELECT * FROM pg_catalog.pg_database"
        cursor.execute(getALLDB)
        ALLDBList = cursor.fetchall()

        for row in ALLDBList:
                if row[1] != "postgres" and row[1].find("template") < 0:
                        print('/opt/pgpro/1c-16/bin/pg_dump -Fc --dbname=postgresql://wms-backup:PAWDSS@ИМЯСЕРВЕРА:5432/' +  row[1] + ' > /mnt/BackUP/' + row[1] + '_' + datenow + '.dump')
                        os.system('/opt/pgpro/1c-16/bin/pg_dump -Fc --dbname=postgresql://wms-backup:СЛОЖНЫЙПАРОЛЬ@EUMOWSQLP051:5432/' +  row[1] + ' > /mnt/BackUP/' + row[1] + '_' + datenow + '.dump')

except (Exception, Error) as error:
        print("Ошибка при работе с PostgreSQL", error)
finally:
        if connection:
                cursor.close()
                connection.close()
                print("Соединение с PostgreSQL закрыто")

```
