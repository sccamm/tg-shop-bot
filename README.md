<div align="center">

# ⭐ TGbuySelStars — Telegram Mini App

**Продажа Telegram Stars и Premium. Пополнение через ЮKassa и TON. Всё внутри Telegram.**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Telegram](https://img.shields.io/badge/Telegram%20WebApp-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=for-the-badge)

</div>

---

## 📖 О проекте

Полноценный **Telegram Mini App (Web App)** для покупки и продажи Telegram Stars, а также оформления подписки Telegram Premium. Работает прямо внутри Telegram — без установки, без переходов в браузер.

### Возможности

- ⭐ **Покупка Stars** — 4 готовых пакета (50 / 100 / 500 / 1000) + произвольное количество
- 👑 **Telegram Premium** — подписки на 3 / 6 / 12 месяцев, отдельные цены для KYC / Non-KYC аккаунтов
- 💰 **Продажа Stars** — обмен звёзд на рубли с моментальным пересчётом курса
- 💳 **Пополнение баланса** — ЮKassa (карта) и TON (крипта с комментарием user_id)
- 📊 **История операций** — список последних транзакций с цветовой кодировкой
- 🔗 **Реферальная система** — ссылка с `?start=r{user_id}` для приглашения друзей
- 🎨 **Современный UI** — glassmorphism, плавные анимации, ripple-эффект на кнопках
- 📱 **Адаптивность** — mobile-first, оптимизировано под Telegram WebApp
- 🌙 **Тёмная тема** — фиолетово-голубой градиент

---

## 🗂 Архитектура

```
tgbuy-stars-miniapp/
├── index.html          — единый файл (SPA, весь UI + логика)
├── (backend)           — API-сервер (не в этом репо)
│   ├── /api/balance       — баланс пользователя
│   ├── /api/transactions  — история
│   ├── /api/buy/stars     — покупка звёзд
│   ├── /api/buy/premium   — покупка Premium
│   ├── /api/sell/stars    — продажа звёзд
│   ├── /api/deposit       — создание платежа (ЮKassa / TON)
│   └── /api/referral      — реферальная статистика
└── README.md
```

Проект — **single-page application (SPA)** в одном HTML-файле. Все экраны (main, buy, premium, deposit, profile, sell) переключаются через JS без перезагрузки страницы.

---

## 💻 Ключевые куски кода

### 1. Инициализация Telegram WebApp (`index.html`)

При загрузке бот получает данные пользователя из `Telegram.WebApp.initDataUnsafe` — ID, username, имя.

```javascript
let tg = window.Telegram.WebApp;
tg.expand();
tg.enableClosingConfirmation();

function initializeApp() {
    const initData = tg.initDataUnsafe;
    userData = {
        id: initData.user?.id || '123456',
        username: initData.user?.username || 'user',
        firstName: initData.user?.first_name || 'User',
        lastName: initData.user?.last_name || ''
    };

    document.getElementById('profileUserId').textContent = userData.id;
    document.getElementById('profileUsername').textContent = '@' + userData.username;
    document.getElementById('userId').textContent = userData.id;

    const referralLink = `https://t.me/your_bot_username?start=r${userData.id}`;
    document.getElementById('referralLink').textContent = referralLink;
}
```

### 2. SPA-роутинг между экранами

Все экраны — отдельные `<div id="...Screen">`, скрытые через класс `.hidden`.

```javascript
function showScreen(screenName) {
    document.querySelectorAll('div[id$="Screen"]').forEach(screen => {
        screen.classList.add('hidden');
    });
    document.getElementById(screenName).classList.remove('hidden');

    document.querySelectorAll('.card').forEach((card, index) => {
        card.style.animation = `fadeIn 0.5s ease-out ${index * 0.1}s both`;
    });
}
```

### 3. Динамическая генерация опций покупки

```javascript
let starPrice = 1.5;

function generateStarOptions() {
    const starOptions = [50, 100, 500, 1000];
    const container = document.getElementById('buyStarsGrid');

    starOptions.forEach(stars => {
        const cost = stars * starPrice;
        const option = document.createElement('div');
        option.className = 'star-option';
        option.onclick = () => selectStars(stars, option);

        option.innerHTML = `
            <div class="star-count">
                <i class="fas fa-star"></i> ${stars}
            </div>
            <div class="star-price">${cost.toFixed(2)} ₽</div>
        `;

        container.appendChild(option);
    });
}
```

### 4. Premium: цены по типу аккаунта

```javascript
const premiumPrices = {
    kyc:    { 3: 299, 6: 549, 12: 899 },
    nonkyc: { 3: 349, 6: 649, 12: 1099 }
};

function selectPremiumType(type) {
    selectedPremiumType = type;
    document.querySelectorAll('.tab').forEach(tab => tab.classList.remove('active'));
    event.target.classList.add('active');
    updatePremiumCost();
}
```

### 5. Продажа звёзд с моментальным пересчётом

```javascript
document.getElementById('sellAmountInput').addEventListener('input', function() {
    const amount = parseInt(this.value) || 0;
    const receiveAmount = amount * starPrice;
    document.getElementById('sellReceiveAmount').textContent =
        receiveAmount.toFixed(2) + ' ₽';
});
```

### 6. Уведомления

```javascript
function showNotification(message, type = 'info') {
    const notification = document.createElement('div');
    notification.className = `notification ${type}`;

    let icon = 'fas fa-info-circle';
    if (type === 'error')   icon = 'fas fa-exclamation-triangle';
    if (type === 'success') icon = 'fas fa-check-circle';
    if (type === 'warning') icon = 'fas fa-exclamation-circle';

    notification.innerHTML = `<i class="${icon}"></i><span>${message}</span>`;
    document.body.appendChild(notification);

    setTimeout(() => notification.remove(), 3000);
}
```

---

## 🎨 Дизайн-система

```css
:root {
    --primary:        #8774E1;
    --secondary:      #34B7F1;
    --premium:        linear-gradient(135deg, #FFD700, #FFA500);
    --success:        #2ECC71;
    --danger:         #E74C3C;
    --background:     linear-gradient(135deg, #1A1A1A, #2D2D2D);
    --card-bg:        rgba(45, 45, 45, 0.7);
    --glass-border:   rgba(255, 255, 255, 0.1);
    --radius-lg:      24px;
    --shadow:         0 8px 32px rgba(0, 0, 0, 0.3);
}
```

**Ключевые эффекты:**
- `backdrop-filter: blur(10px)` — стеклянные карточки
- Ripple-анимация на кнопках
- Каскадный `fadeIn` при переходах
- `pulse` на плавающей кнопке «+»

---

## 🛠 Стек технологий

| Компонент | Технология |
|---|---|
| Разметка | HTML5 |
| Стили | CSS3 (переменные, grid, flex, градиенты, backdrop-filter) |
| Логика | Vanilla JavaScript (ES6+) |
| Иконки | Font Awesome 6.4 |
| Платформа | Telegram WebApp API |

**Без фреймворков.** Чистый HTML + CSS + JS.

---

## 🚀 Развёртывание

### Как Telegram Mini App

1. Загрузи `index.html` на HTTPS-хостинг (Vercel / Netlify)
2. В **@BotFather** → `/newapp` → укажи URL
3. Пользователи открывают приложение через кнопку в боте

### Локально

```bash
python -m http.server 8000
# → http://localhost:8000
```

### Что заменить

| Параметр | На что заменить |
|---|---|
| `your_bot_username` | username твоего бота |
| `starPrice` | актуальный курс 1 звезды |
| `premiumPrices` | актуальные цены Premium |
| `tonAddress` | реальный TON-кошелёк |
| `setTimeout(...)` | `fetch('/api/...')` к бэкенду |

### Подключение бэкенда

```javascript
async function processBuyStars() {
    const recipient = document.getElementById('recipientInput').value;
    if (!recipient || selectedStars === 0) {
        showNotification('Заполните все поля', 'error');
        return;
    }

    try {
        const res = await fetch('/api/buy/stars', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-Telegram-Init-Data': tg.initData
            },
            body: JSON.stringify({
                recipient,
                amount: selectedStars,
                user_id: userData.id
            })
        });

        const data = await res.json();
        if (data.ok) {
            showNotification(`Куплено ${selectedStars} звёзд!`, 'success');
            showScreen('mainScreen');
            loadUserData();
        } else {
            showNotification(data.error || 'Ошибка', 'error');
        }
    } catch (e) {
        showNotification('Ошибка соединения', 'error');
    }
}
```

> ⚠️ **Важно:** Всегда проверяй `tg.initData` на бэкенде — это HMAC-подпись от Telegram.

---

## 📸 Скриншоты

### Главный экран
<img width="492" height="1024" alt="efc8fe1b-ffb5-41c6-9c9c-fde9f3afa9a0" src="https://github.com/user-attachments/assets/85aed70a-98d0-42dc-bc92-57cc2bdb90ea" />

### Профиль
<img width="483" height="1024" alt="58ad9db2-9e24-47b3-b725-d539e236a500" src="https://github.com/user-attachments/assets/4a298994-c558-4d79-b3ea-22864b526dbe" />

### Пополнение баланса
<img width="482" height="1024" alt="525b2b72-33c5-4e78-9b9f-19d4fee80752" src="https://github.com/user-attachments/assets/38b9e7ec-5a1f-419b-9a43-431d58487e1c" />
