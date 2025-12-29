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

# Теперь восстанавливаем уже из этого места
sudo -u postgres pg_restore \
  -d wms-test \
  --clean --if-exists \
  --no-owner --no-privileges \
  -v \
  /var/tmp/pg_restore/wms-upp_2025_12_29_163001.dump \
  > /home/ikoshcheev/restore_log_20251229.txt 2>&1
```
