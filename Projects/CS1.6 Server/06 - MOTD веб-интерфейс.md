# MOTD — веб-интерфейс

## Техническая база

| Параметр | Значение |
|----------|----------|
| **Движок** | Trident (Internet Explorer) в старых сборках, WebView в новых клиентах |
| **HTML** | HTML 4.01 / XHTML 1.0 |
| **CSS** | CSS 2.1, частично CSS3 (анимации, трансформации) |
| **JavaScript** | **ES5** (нет const/let, нет arrow functions, нет шаблонных строк) |
| **HTTP** | **XMLHttpRequest** (fetch НЕ работает в Trident!) |
| **Ограничения** | Нет доступа к локальной ФС, запрещены межсайтовые запросы (CORS) |
| **Размер окна** | 800×600 пикселей |

> **КРИТИЧЕСКИ ВАЖНО:** Клиент CS 1.6 использует Trident (IE7-IE11). **fetch() НЕ работает!** Все HTTP-запросы через **XMLHttpRequest**.

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

## Пример кода главной страницы (ES5)

```html
<!-- menu.html -->
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>CS: Economic Warfare</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div id="app">
        <nav id="tabs">
            <button class="tab active" data-page="home">🏠 Главная</button>
            <button class="tab" data-page="war-map">🗺️ Карта войны</button>
            <button class="tab" data-page="general">👑 Генерал-штаб</button>
            <button class="tab" data-page="clans">⚔️ Кланы</button>
            <button class="tab" data-page="business">🏢 Бизнес</button>
            <button class="tab" data-page="exchange">📊 Биржа</button>
            <button class="tab" data-page="contracts">📦 Контракты</button>
            <button class="tab" data-page="cases">🎁 Кейсы</button>
            <button class="tab" data-page="profile">👤 Профиль</button>
        </nav>
        <div id="content"></div>
    </div>
    <script src="js/main.js"></script>
</body>
</html>
```

---

## JavaScript (ES5 + XMLHttpRequest)

```javascript
// js/main.js - ES5 совместимый
(function() {
    // Получаем SteamID из URL
    var params = window.location.search.substring(1).split('&');
    var steamid = '';
    for (var i = 0; i < params.length; i++) {
        var pair = params[i].split('=');
        if (pair[0] === 'steamid') {
            steamid = pair[1];
        }
    }
    
    // Загрузка страницы
    function loadPage(page) {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'pages/' + page + '.html', true);
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4 && xhr.status === 200) {
                document.getElementById('content').innerHTML = xhr.responseText;
                initPage(page);
            }
        };
        xhr.send();
    }
    
    // API запрос (XMLHttpRequest вместо fetch!)
    function apiRequest(endpoint, callback) {
        var xhr = new XMLHttpRequest();
        xhr.open('GET', '/api/' + endpoint + '?steamid=' + steamid, true);
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4 && xhr.status === 200) {
                var data = JSON.parse(xhr.responseText);
                callback(data);
            }
        };
        xhr.send();
    }
    
    // Инициализация табов
    var tabs = document.querySelectorAll('.tab');
    for (var i = 0; i < tabs.length; i++) {
        tabs[i].addEventListener('click', function() {
            var page = this.getAttribute('data-page');
            loadPage(page);
            
            // Обновление активного таба
            var allTabs = document.querySelectorAll('.tab');
            for (var j = 0; j < allTabs.length; j++) {
                allTabs[j].classList.remove('active');
            }
            this.classList.add('active');
        });
    }
    
    // Загружаем главную страницу
    loadPage('home');
})();
```

---

## Canvas-карта (ES5)

```javascript
// js/map.js - ES5 совместимый
var WarMap = {
    canvas: null,
    ctx: null,
    
    init: function() {
        this.canvas = document.getElementById('warCanvas');
        if (!this.canvas) return;
        this.ctx = this.canvas.getContext('2d');
        this.draw();
        var self = this;
        setInterval(function() { self.draw(); }, 5000);
    },
    
    draw: function() {
        var self = this;
        var xhr = new XMLHttpRequest();
        xhr.open('GET', '/api/war?steamid=' + steamid, true);
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4 && xhr.status === 200) {
                var data = JSON.parse(xhr.responseText);
                self.render(data);
            }
        };
        xhr.send();
    },
    
    render: function(data) {
        this.ctx.clearRect(0, 0, 800, 600);
        
        // Линии снабжения
        for (var i = 0; i < data.lines.length; i++) {
            var l = data.lines[i];
            this.ctx.beginPath();
            this.ctx.moveTo(l.x1, l.y1);
            this.ctx.lineTo(l.x2, l.y2);
            this.ctx.strokeStyle = '#666';
            this.ctx.lineWidth = 2;
            this.ctx.stroke();
        }
        
        // Города
        for (var i = 0; i < data.cities.length; i++) {
            var c = data.cities[i];
            this.ctx.fillStyle = c.owner === 'blue' ? '#2e7d32' : '#d32f2f';
            this.ctx.beginPath();
            this.ctx.arc(c.x, c.y, 15, 0, 2 * Math.PI);
            this.ctx.fill();
            this.ctx.fillStyle = '#fff';
            this.ctx.font = '10px Arial';
            this.ctx.fillText(c.name, c.x - 20, c.y - 20);
        }
        
        // Штурмовые группы (AT)
        for (var i = 0; i < data.ATs.length; i++) {
            var at = data.ATs[i];
            this.ctx.font = '20px Arial';
            this.ctx.fillStyle = at.owner === 'blue' ? '#4fc3f7' : '#ef9a9a';
            this.ctx.fillText(at.icon, at.x, at.y + 6);
        }
    }
};
```

---

## CSS (ES5 совместимый)

```css
/* css/style.css */
body {
    margin: 0;
    padding: 0;
    font-family: Arial, sans-serif;
    background: #1a1a2e;
    color: #fff;
    width: 800px;
    height: 600px;
    overflow: hidden;
}

#app {
    display: flex;
    flex-direction: column;
    height: 100%;
}

#tabs {
    display: flex;
    background: #16213e;
    padding: 5px;
    border-bottom: 2px solid #0f3460;
}

.tab {
    flex: 1;
    padding: 8px 4px;
    background: #0f3460;
    border: none;
    color: #fff;
    cursor: pointer;
    font-size: 11px;
    margin: 0 2px;
    border-radius: 4px 4px 0 0;
}

.tab:hover {
    background: #533483;
}

.tab.active {
    background: #e94560;
}

#content {
    flex: 1;
    padding: 15px;
    overflow-y: auto;
    background: #1a1a2e;
}

/* Таблицы */
table {
    width: 100%;
    border-collapse: collapse;
    margin: 10px 0;
}

th, td {
    padding: 8px;
    border: 1px solid #0f3460;
    text-align: left;
}

th {
    background: #16213e;
    color: #e94560;
}

/* Кнопки */
.btn {
    padding: 8px 16px;
    background: #e94560;
    border: none;
    color: #fff;
    cursor: pointer;
    border-radius: 4px;
}

.btn:hover {
    background: #533483;
}

/* Прогресс-бары */
.progress {
    width: 100%;
    height: 20px;
    background: #16213e;
    border-radius: 10px;
    overflow: hidden;
    margin: 5px 0;
}

.progress-bar {
    height: 100%;
    background: linear-gradient(90deg, #e94560, #533483);
    transition: width 0.3s;
}
```

---

## Связанные заметки

- [[05 - Техническая архитектура]]
- [[08 - Плагины AMXX]]
- [[03 - Игровые механики]]
- [[11 - Дорожная карта]]
