# TODO — День 1

**Дата:** 01.09.2026
**Сервер:** Orange Pi Zero 1GB (192.168.1.7)

---

## Выполнено

- [x] Установка Armbian обновлена до 26.8.3
- [x] Docker 20.10.24 установлен (VFS driver, без iptables)
- [x] Структура проекта создана: `cs-economic-warfare/`
- [x] `docker-compose.yml` — 4 сервиса (cs16, api, nginx, bot)
- [x] `server/cstrike/server.cfg` — конфиг сервера
- [x] `data/schema.sql` — SQLite схема (players, businesses, cities, clans, skins, contracts)
- [x] `api/app.py` — Flask REST API (7 эндпоинтов)
- [x] `bot/bot.py` — Telegram-бот (aiogram, 7 команд)
- [x] `web/menu.html` — MOTD интерфейс (ES5, XMLHttpRequest)
- [x] `web/js/main.js` — JavaScript (ES5 совместимый)
- [x] `web/css/style.css` — стили (800x600)
- [x] `deploy.sh` — скрипт деплоя на сервер
- [x] `setup.sh` — первичная настройка OPi
- [x] GitHub репозиторий: https://github.com/NickSam3/cs-economic-warfare

---

## Проблемы

| Проблема | Решение |
|----------|---------|
| Docker bridge не работает (нет iptables/nft) | Запуск с `iptables:false, bridge:none` |
| Docker VFS сожрал RAM → сервер упал | Не использовать VFS для тяжёлых образов |
| SSH зависает после Docker OOM | Нужна ручная перезагрузка OPi |

---

## Следующий шаг

1. **Перезагрузить Orange Pi** (выдернуть питание)
2. Зайти по SSH
3. Клонировать репозиторий: `git clone https://github.com/NickSam3/cs-economic-warfare.git`
4. Запустить `bash deploy.sh`
5. Проверить: `curl http://localhost/api/health`
