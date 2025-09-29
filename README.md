# мониторинга процесса `test`

1) Создадим `/usr/local/bin/test_monitor.sh`:

```bash
#!/bin/bash

LOGFILE="/var/log/monitoring.log"
PIDFILE="/run/test_monitor.pid"
API_URL="https://test.com/monitoring/test/api"

# Получаем PID процесса test по точному имени
PID=$(pgrep -x test)

if [ -z "$PID" ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') | Процесс test не запущен" >> "$LOGFILE"
    exit 0
fi

# Проверка на перезапуск (смена PID)
if [ -f "$PIDFILE" ]; then
    OLD_PID=$(cat "$PIDFILE" 2>/dev/null || true)
    if [ -n "$OLD_PID" ] && [ "$OLD_PID" != "$PID" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') | Процесс test перезапущен (PID изменился с $OLD_PID на $PID)" >> "$LOGFILE"
    fi
fi
echo "$PID" > "$PIDFILE"

# Проверка доступности API (таймаут 5 секунд)
if ! curl -sS --max-time 5 "$API_URL" > /dev/null; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') | API недоступен" >> "$LOGFILE"
fi


```

2) Создаем Юнит systemd для автозапуска. Создаём `/etc/systemd/system/test-monitor.service`:

```bash
[Unit]
Description=Monitoring test process and API

[Service]
Type=oneshot
ExecStart=/usr/local/bin/test_monitor.sh
```
3) Настройка таймера systemd.  Создаем `/etc/systemd/system/test-monitor.timer`:

 ```bash
[Unit]
Description=Run test-monitor every minute

[Timer]
OnBootSec=1min
OnUnitActiveSec=1min
AccuracySec=10s
Unit=test-monitor.service

[Install]
WantedBy=timers.target
```
4) Установка и активация

 ```bash
# Установка прав и подготовка лог-файла
sudo install -m 0755 /usr/local/bin/test_monitor.sh /usr/local/bin/test_monitor.sh
sudo touch /var/log/monitoring.log
sudo chmod 0644 /var/log/monitoring.log

# Активация таймера
sudo systemctl daemon-reload
sudo systemctl enable --now test-monitor.timer
```


