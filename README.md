<div align="center">

# 🤖 Telegram Shop Bot\Web3

**Бот-магазин Telegram Stars с каталогом, корзиной и админ-панелью**

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![aiogram](https://img.shields.io/badge/aiogram-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)

</div>

---

## 📖 О проекте

Web3 бот-магазин Telegram Stars с оплатой, каталогом товаров и админ-панелью прямо в Telegram.

### Возможности

- 🛍 **Каталог** с категориями и поиском
- 🛒 **Корзина** и оформление заказа
- 💳 **Оплата** через ЮKassa / CryptoBot
- 📊 **Админ-панель** в Telegram: статистика, товары, рассылка
- 🗄 **SQLite** с миграциями

---
# ВСЕ НИЖЕ ПОКАЗАННО ДЛЯ ПРИМЕРА ТАК КАК ОНО ВЫПОЛНЕНО НА ЗАКАЗ И НА ДАННЫЙ МОМЕНТ АКТИВНО РАБОТАЕТ

## 🚀 Установка

```bash
git clone https://github.com/sccamm/tg-shop-bot.git
cd tg-shop-bot
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
# отредактируй .env: BOT_TOKEN, ADMIN_IDS
python bot.py
```

---

## 🗂 Структура

```
tg-shop-bot/
├── bot.py              — точка входа
├── config.py           — загрузка .env
├── database.py         — работа с SQLite
├── handlers/
│   ├── start.py
│   ├── catalog.py
│   ├── cart.py
│   └── admin.py
├── keyboards/
│   └── inline.py
├── requirements.txt
└── .env.example
```

---

## 💻 Пример кода

**`database.py` — работа с SQLite**

```python
import aiosqlite
from config import DB_PATH


async def init_db():
    async with aiosqlite.connect(DB_PATH) as db:
        await db.execute("""
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price INTEGER NOT NULL,
                category TEXT NOT NULL,
                stock INTEGER DEFAULT 0
            )
        """)
        await db.execute("""
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_id INTEGER NOT NULL,
                product_id INTEGER NOT NULL,
                amount INTEGER NOT NULL,
                status TEXT DEFAULT 'pending',
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        await db.commit()


async def get_products(category: str = None):
    async with aiosqlite.connect(DB_PATH) as db:
        if category:
            q = "SELECT id, name, price, stock FROM products WHERE category = ?"
            args = (category,)
        else:
            q = "SELECT id, name, price, stock FROM products"
            args = ()
        async with db.execute(q, args) as cur:
            rows = await cur.fetchall()
            return [dict(zip(("id","name","price","stock"), r)) for r in rows]


async def create_order(user_id: int, product_id: int, amount: int) -> int:
    async with aiosqlite.connect(DB_PATH) as db:
        cur = await db.execute(
            "INSERT INTO orders (user_id, product_id, amount) VALUES (?, ?, ?)",
            (user_id, product_id, amount)
        )
        await db.commit()
        return cur.lastrowid
```

**`handlers/catalog.py` — каталог с инлайн-кнопками**

```python
from aiogram import Router, F
from aiogram.types import CallbackQuery, InlineKeyboardButton
from aiogram.utils.keyboard import InlineKeyboardBuilder

import database as db

router = Router()


@router.callback_query(F.data == "catalog")
async def show_catalog(call: CallbackQuery):
    products = await db.get_products()
    kb = InlineKeyboardBuilder()
    for p in products:
        kb.button(
            text=f"{p['name']} — {p['price']}₽",
            callback_data=f"buy:{p['id']}"
        )
    kb.adjust(1)
    kb.row(InlineKeyboardButton(text="⬅️ Назад", callback_data="main_menu"))

    await call.message.edit_text(
        "🛍 <b>Каталог товаров</b>\nВыбери товар:",
        reply_markup=kb.as_markup(),
        parse_mode="HTML"
    )


@router.callback_query(F.data.startswith("buy:"))
async def buy_product(call: CallbackQuery):
    product_id = int(call.data.split(":")[1])
    products = await db.get_products()
    product = next((p for p in products if p["id"] == product_id), None)
    if not product:
        return await call.answer("Товар не найден", show_alert=True)

    order_id = await db.create_order(call.from_user.id, product_id, product["price"])
    await call.message.edit_text(
        f"✅ Заказ <b>#{order_id}</b> создан\n"
        f"Товар: <b>{product['name']}</b>\n"
        f"Сумма: <b>{product['price']}₽</b>\n\n"
        f"Перейди к оплате: /pay_{order_id}",
        parse_mode="HTML"
    )
```

**`bot.py` — точка входа**

```python
import asyncio
import logging
from aiogram import Bot, Dispatcher
from aiogram.fsm.storage.memory import MemoryStorage

from config import BOT_TOKEN
import database as db
from handlers import start, catalog, cart, admin

logging.basicConfig(level=logging.INFO)


async def main():
    await db.init_db()

    bot = Bot(token=BOT_TOKEN)
    dp = Dispatcher(storage=MemoryStorage())

    dp.include_router(start.router)
    dp.include_router(catalog.router)
    dp.include_router(cart.router)
    dp.include_router(admin.router)

    await bot.delete_webhook(drop_pending_updates=True)
    await dp.start_polling(bot)


if __name__ == "__main__":
    asyncio.run(main())
```

---

## 🗄 Схема БД

| Таблица | Поля |
|---|---|
| `products` | id, name, price, category, stock |
| `orders` | id, user_id, product_id, amount, status, created_at |
| `users` | id, username, balance, is_admin |

---

## 📸 Скриншоты

### Главный экран
![Главный экран](<img width="492" height="1024" alt="efc8fe1b-ffb5-41c6-9c9c-fde9f3afa9a0" src="https://github.com/user-attachments/assets/85aed70a-98d0-42dc-bc92-57cc2bdb90ea" />)

### Профиль
![Профиль](<img width="483" height="1024" alt="58ad9db2-9e24-47b3-b725-d539e236a500" src="https://github.com/user-attachments/assets/4a298994-c558-4d79-b3ea-22864b526dbe" />)

### Пополнение баланса
![Оформление](<img width="482" height="1024" alt="525b2b72-33c5-4e78-9b9f-19d4fee80752" src="https://github.com/user-attachments/assets/38b9e7ec-5a1f-419b-9a43-431d58487e1c" />
)
