# MOTD — веб-интерфейс

## Техническая база

| Параметр | Значение |
|----------|----------|
| **Движок** | Trident (Internet Explorer) в старых сборках, WebView в новых клиентах |
| **HTML** | HTML 4.01 / XHTML 1.0 |
| **CSS** | CSS 2.1, частично CSS3 (анимации, трансформации) |
| **JavaScript** | ES5, поддерживается XMLHttpRequest (AJAX), fetch, Canvas, addEventListener |
| **Ограничения** | Нет доступа к локальной ФС, запрещены межсайтовые запросы (CORS) |
| **Размер окна** | 800×600 пикселей |

---

## Открытие MOTD (AMXX)

```pawn
public cmdOpenMenu(id) {
    new szSteamID[32];
    get_user_authid(id, szSteamID, charsmax(szSteamID));
    new szUrl[256];
    formatex(szUrl, charsmax(szUrl),
        "http://cs-economic.duckdns.org/menu.html?steamid=%s", szSteamID);
    show_motd(id, szUrl, "Economic Warfare");
    return PLUGIN_HANDLED;
}
```

---

## Структура вкладок

| Вкладка | Иконка | Назначение |
|---------|--------|------------|
| **Главная** | 🏠 | Приветствие, баланс, ежедневный бонус, статус войны |
| **Карта войны** | 🗺️ | Canvas-карта с городами, линиями, AT (анимация) |
| **Генерал-штаб** | 👑 | Управление AT (генералы) |
| **Кланы** | ⚔️ | Информация о клане, управление, войны |
| **Бизнес** | 🏢 | Список бизнесов, доход, улучшение/продажа |
| **Биржа** | 📊 | График курса Coins, торги активами |
| **Контракты** | 📦 | Ежедневные/еженедельные задания |
| **Кейсы** | 🎁 | Открытие кейсов с анимацией |
| **Профиль** | 👤 | Статистика, достижения, настройки |

---

## Пример кода карты (Canvas)

```javascript
const canvas = document.getElementById('warCanvas');
const ctx = canvas.getContext('2d');

async function drawMap() {
    const data = await fetch('/api/war').then(r => r.json());
    ctx.clearRect(0, 0, 800, 600);
    // Линии снабжения
    data.lines.forEach(l => {
        ctx.beginPath();
        ctx.moveTo(l.x1, l.y1);
        ctx.lineTo(l.x2, l.y2);
        ctx.strokeStyle = '#666';
        ctx.stroke();
    });
    // Города
    data.cities.forEach(c => {
        ctx.fillStyle = c.owner === 'blue' ? '#2e7d32' : '#d32f2f';
        ctx.beginPath();
        ctx.arc(c.x, c.y, 15, 0, 2*Math.PI);
        ctx.fill();
        ctx.fillStyle = '#fff';
        ctx.fillText(c.name, c.x-20, c.y-30);
    });
    // AT (анимируются)
    data.ATs.forEach(at => {
        ctx.font = '20px sans-serif';
        ctx.fillStyle = at.owner === 'blue' ? '#4fc3f7' : '#ef9a9a';
        ctx.fillText(at.icon, at.x, at.y+6);
    });
}
setInterval(drawMap, 5000);
```

---

## Связанные заметки

- [[05 - Техническая архитектура]]
- [[08 - Плагины AMXX]]
- [[03 - Игровые механики]]
