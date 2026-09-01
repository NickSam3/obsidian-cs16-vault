# Плагины AMXX — полное руководство

## Структура плагина

```pawn
#include <amxmodx>
#include <amxmisc>
#include <cstrike>
#include <sqlx>

#define PLUGIN "Economic Warfare Core"
#define VERSION "1.0"
#define AUTHOR "Dev"

new g_iCoins[33];
new Handle:g_SqlTuple;

public plugin_init() {
    register_plugin(PLUGIN, VERSION, AUTHOR);
    register_clcmd("say /menu", "cmdOpenMenu");
    register_clcmd("say !menu", "cmdOpenMenu");
    register_event("DeathMsg", "eventDeath", "a");
    g_SqlTuple = SQL_MakeDbTuple("localhost", "root", "", "economic_warfare");
}
```

---

## Компиляция и установка

1. Поместить `.sma` в `addons/amxmodx/scripting/`
2. Запустить компилятор: `amxxpc.exe plugin.sma`
3. Скопировать `.amxx` в `addons/amxmodx/plugins/`
4. Добавить в `configs/plugins.ini`:

```
economic_core.amxx
war_core.amxx
clan_system.amxx
skin_system.amxx
business_system.amxx
api_bridge.amxx
```

---

## Основные плагины проекта

| Плагин | Назначение | Интеграция |
|--------|-----------|------------|
| **Economic Core** | Coins, бизнесы, пассивный доход | SQLite/MySQL, MOTD |
| **War Core** | Отправка результатов боёв на API | HTTP (http2.amxx) |
| **Clan System** | Создание кланов, управление, войны | SQLite/MySQL |
| **Skin System** | Кастомные скины, инвентарь | SQLite/MySQL, MOTD |
| **Business System** | Пассивный доход от бизнесов | SQLite/MySQL |
| **API Bridge** | Мост между AMXX и Python API | HTTP (http2.amxx) |
| **Telegram Bridge** | Получение команд от бота, начисление Stars | HTTP API |

---

## HTTP-запросы из AMXX

### Важно: AmxxEasyHttp НЕ подтверждён для ARM!

Используйте **http2.amxx** или **netchan** плагин:

```pawn
// addons/amxmodx/scripting/api_bridge.sma
#include <amxmodx>
#include <amxmisc>
#include <http2>

#define PLUGIN "Economic Warfare - API Bridge"
#define VERSION "1.0"
#define AUTHOR "CS:EW Team"

new const API_URL[] = "http://192.168.1.7:5000";

public plugin_init() {
    register_plugin(PLUGIN, VERSION, AUTHOR);
    register_event("DeathMsg", "onPlayerDeath", "a");
}

public onPlayerDeath() {
    new killer = read_data(1);
    new victim = read_data(2);
    new headshot = read_data(3);
    
    if(killer == victim || !is_user_connected(killer)) return;
    
    new szKillerID[32], szVictimID[32];
    get_user_authid(killer, szKillerID, charsmax(szKillerID));
    get_user_authid(victim, szVictimID, charsmax(szVictimID));
    
    new szData[256];
    formatex(szData, charsmax(szData),
        "winner=%s&loser=%s&points=%d&headshot=%d",
        szKillerID, szVictimID, headshot ? 15 : 10, headshot);
    
    // HTTP POST через http2.amxx
    new szUrl[256];
    formatex(szUrl, charsmax(szUrl), "%s/api/battle/result", API_URL);
    
    http2_post(szUrl, szData, "onBattleResult");
}

public onBattleResult(success, const data[]) {
    if(success) {
        server_print("[CS:EW] Battle result sent successfully");
    } else {
        server_print("[CS:EW] Failed to send battle result");
    }
}
```

### Альтернатива: netchan плагин

```pawn
#include <amxmodx>
#include <netchan>

// HTTP POST запрос
sendPostRequest(endpoint[], data[]) {
    new szUrl[256];
    formatex(szUrl, charsmax(szUrl), "%s%s", API_URL, endpoint);
    
    new sock = socket_open(szUrl, 80, SOCKET_TCP, err);
    if(err) {
        server_print("[CS:EW] Socket error: %d", err);
        return;
    }
    
    new szRequest[512];
    formatex(szRequest, charsmax(szRequest),
        "POST %s HTTP/1.1^nHost: %s^nContent-Type: application/x-www-form-urlencoded^nContent-Length: %d^n^n%s",
        endpoint, "192.168.1.7", strlen(data), data);
    
    socket_write(sock, szRequest, strlen(szRequest));
    socket_close(sock);
}
```

---

## Установка HTTP-плагина

```bash
# Скачать http2.amxx для ARM
cd /opt/cs-economic/data/server/cstrike/addons/amxmodx
wget https://github.com/Arkhan/ReHLDS/releases/download/http2/http2_linux_arm.tar.gz
tar xzf http2_linux_arm.tar.gz

# Или скомпилировать из исходников
cd addons/amxmodx/scripting
wget https://github.com/alliedmodders/amxmodx/archive/master.zip
# Скомпилировать http2.sma
```

---

## Связанные заметки

- [[05 - Техническая архитектура]]
- [[06 - MOTD веб-интерфейс]]
- [[07 - Telegram-бот]]
- [[11 - Дорожная карта]]
