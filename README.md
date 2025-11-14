### Что нужно сделать на старте?
###### Админу:
1. Поменять пароль ssh (подключиться к vulnbox и поменять пароль через passwd)
2. Cбилдить сервисы: ```
```bash
docker compose up -d
```
3. (после внесений изменений) пересобрать сервис:
```bash
docker compose restart
```
4. Список запущенных сервисов:
```bash
docker compose ps
```
###### Остальным:
1. Скопировать папки с сервисами на свою машину:
```bash
scp -r root@<SERVER_IP>:/path/to/services ~/innoctf/
```
2. Начать искать уязвимости.
3. Запустить `codex`. Начальный промпт:
```
i am developing my own A/D game. the services are located in this dir. i am going to test my A/D on you, you need to find vulnerabilites in these services and write exploits for them. exploits should print flags, flag format is [A-Z0-9]{31}=
```
Если эксплоиты не работают, можно дать ИИ полный доступ через `/approvals` и попросить протестировать эксплоиты и исправить в них ошибки:
```
test your exploits here on my ip <VULN_BOX_IP>. i gave you access to network
```


### Сетевой мониторинг
1. **Packmate** https://gitlab.com/packmate/starter
2. **Альтернатива** - wireshark:
```bash
sshpass -p<PASSWORD> ssh root@<IP> tcpdump -i vulnbox -U -s0 -w - 'not port 22' | wireshark -k -i -
```

### Как засылать флаги
##### 1. Обычный способ
curl:
```bash
curl -s -H 'X-Team-Token: your_secret_token' -X PUT -d '["PNFP4DKBOV6BTYL9YFGBQ9006582ADC=", "STH5LK9R9OMGXOV4E06YZD71F746F53=", "0I7DUCYPX8UB2HP6D6UGN86BA26F2FE=", "PTK3DAGZ6XU4LPETXJTN7CE30EC0B54="]' http://10.10.10.10/flags

```
Python:
```python
import requests, json

response = requests.put('http://<BOARD_IP>/flags', headers={'X-Team-Token': '<token>'}, data=json.dumps(flags));
```
#### 2. Через ферму (если она поднята):
Нужно использовать `start_sploit` из репозитория фермы для запуска эксплоита:
```bash
python3 start_sploit.py exploit.py -u http://<FARM_IP>
```
Пример эксплоита:
```python
#!/usr/bin/env python3
"""
Бланковый сплойт для S4DFarm

Запуск:
    python3 start_sploit.py sploit_template.py \
        --server-url http://<IP_СЕРВЕРА>:5137 \
        --server-pass 1234

Требования:
    - Shebang в первой строке (уже есть)
    - Принимает IP команды как первый аргумент (sys.argv[1])
    - Выводит флаги в stdout/stderr с flush=True
    - Флаги должны соответствовать формату из конфига сервера
"""

import sys

if len(sys.argv) < 2:
    print(f'Usage: {sys.argv[0]} <target>', flush=True)
    sys.exit(1)

target_ip = sys.argv[1]

# TODO: Реализуйте вашу логику атаки здесь
# Подключитесь к сервису команды, найдите уязвимость, получите флаги

try:
    # Пример: подключение к сервису
    import requests
    response = requests.get(f'http://{target_ip}:8080/api/flags', timeout=5)
    flags = response.json().get('flags', [])
    
    # Пример: вывод флагов
    for flag in flags:
		print(flag, flush=True)
    
    pass
except Exception:
    # Обрабатывайте ошибки тихо, чтобы не спамить логи
    pass

```
