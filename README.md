# bash

#!/bin/bash

LOGFILE="/var/log/monitoring.log"
PIDFILE="/var/run/test_monitor.pid"
API_URL="https://test.com/monitoring/test/api"

echo "Старт мониторинга..." >> "$LOGFILE"

OLD_PID=""

while true; do
    # Проверяем процесс
    PID=$(pgrep -x test)

    if [ -z "$PID" ]; then
        echo "$(date '+%Y-%m-%d %H:%M:%S') | Процесс test не запущен" >> "$LOGFILE"
    else
        # Проверка на перезапуск
        if [ "$OLD_PID" != "" ] && [ "$OLD_PID" != "$PID" ]; then
            echo "$(date '+%Y-%m-%d %H:%M:%S') | Процесс test перезапущен (PID изменился с $OLD_PID на $PID)" >> "$LOGFILE"
        fi
        OLD_PID=$PID

        # Проверка API
        curl -s --max-time 5 "$API_URL" > /dev/null
        if [ $? -ne 0 ]; then
            echo "$(date '+%Y-%m-%d %H:%M:%S') | API недоступен" >> "$LOGFILE"
        fi
    fi

    sleep 60
done
