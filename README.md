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
