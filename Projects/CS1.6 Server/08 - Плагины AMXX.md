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
```

---

## Основные плагины проекта

| Плагин | Назначение | Интеграция |
|--------|-----------|------------|
| **Economic Core** | Coins, бизнесы, пассивный доход | MySQL, MOTD |
| **War Core** | Отправка результатов боёв на API | HTTP (AmxxEasyHttp) |
| **Clan System** | Создание кланов, управление, войны | MySQL |
| **Skin System** | Кастомные скины, инвентарь | MySQL, MOTD |
| **Telegram Bridge** | Получение команд от бота, начисление Stars | HTTP API |

---

## HTTP-запросы из AMXX (AmxxEasyHttp)

```pawn
#include <ezhttp>

public sendBattleResult(winner, looser, points) {
    new szUrl[256], szData[128];
    formatex(szUrl, charsmax(szUrl),
        "http://cs-economic.duckdns.org/api/battle/result");
    formatex(szData, charsmax(szData),
        "winner=%d&looser=%d&points=%d", winner, looser, points);
    
    new EzHttpOptions:opts = ezhttp_create_options();
    ezhttp_option_set_header(opts, "Content-Type", "application/x-www-form-urlencoded");
    ezhttp_option_set_body(opts, szData);
    ezhttp_post(szUrl, "httpCallback", opts);
}
```

---

## Связанные заметки

- [[05 - Техническая архитектура]]
- [[06 - MOTD веб-интерфейс]]
- [[07 - Telegram-бот]]
