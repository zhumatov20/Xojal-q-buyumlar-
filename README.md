# Xojal-q-buyumlar-
import asyncio
import logging
from aiogram import Bot, Dispatcher, types
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton, InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.filters import Command

# Bot tokenini shu yerda yozing
TOKEN = "8189877832:AAG2Si5_Eh8LRzOqOXVrqLm0PQezDkGNk50"
ADMIN_ID = 7113124338  # Admin ID

# Bot va Dispatcher yaratish
bot = Bot(token=TOKEN)
dp = Dispatcher()

# Asosiy menyu tugmalari
main_menu = ReplyKeyboardMarkup(keyboard=[
    [KeyboardButton(text="🛍 Mahsulotlar")],
    [KeyboardButton(text="📞 Aloqa"), KeyboardButton(text="ℹ️ Ma’lumot")],
    [KeyboardButton(text="💳 To‘lov"), KeyboardButton(text="📦 Buyurtmalarim")]
], resize_keyboard=True)

# Mahsulotlar va rasmlar
products = {
    "Qazan-tabaq": [
        {"name": "Qazan 5L", "price": 100000, "image": "https://example.com/qazan.jpg"},
        {"name": "Tabaq 3L", "price": 80000, "image": "https://example.com/tabaq.jpg"}
    ],
    "Plastmassa buyímlar": [
        {"name": "Chelak 10L", "price": 30000, "image": "https://example.com/chelak.jpg"},
        {"name": "Hovuz 50L", "price": 150000, "image": "https://example.com/hovuz.jpg"}
    ],
    "Elektr ha’m qosímsha buyumlar": [
        {"name": "Choynak", "price": 200000, "image": "https://example.com/choynak.jpg"},
        {"name": "Lampochka", "price": 20000, "image": "https://example.com/lamp.jpg"}
    ]
}

# Kategoriyalar tugmalari
product_menu = ReplyKeyboardMarkup(
    keyboard=[[KeyboardButton(text=category)] for category in products.keys()] +
    [[KeyboardButton(text="⬅️ Orqaga")]],
    resize_keyboard=True
)

# 🔹 Start komandasi
@dp.message(Command("start"))
async def send_welcome(message: types.Message):
    await message.answer("Xojalíq buyumlarí botına xos kelipsiz!", reply_markup=main_menu)

# 🔹 Mahsulot kategoriyalarini ko‘rsatish
@dp.message(lambda message: message.text == "🛍 Mahsulotlar")
async def show_products(message: types.Message):
    await message.answer("Kerakli kategoriyani tanla:", reply_markup=product_menu)

# 🔹 Mahsulotlarni rasm bilan chiqarish
@dp.message(lambda message: message.text in products.keys())
async def show_product_list(message: types.Message):
    category = message.text
    for product in products[category]:
        caption = f"📦 {product['name']}\n💰 Narx: {product['price']} so‘m"
        await bot.send_photo(message.chat.id, product["image"], caption=caption)

# 🔹 Narxlarni tugma bosganda ko‘rsatish (Inline tugmalar)
@dp.message(lambda message: message.text in [p["name"] for cat in products.values() for p in cat])
async def show_price(message: types.Message):
    for category, items in products.items():
        for product in items:
            if message.text == product["name"]:
                inline_kb = InlineKeyboardMarkup(
                    inline_keyboard=[[InlineKeyboardButton(text="💳 Sotib olish", callback_data=f"buy_{product['name']}")]]
                )
                await message.answer(f"💰 Narx: {product['price']} so‘m", reply_markup=inline_kb)

# 🔹 To‘lov tizimi tugmalari (Click & Payme)
@dp.message(lambda message: message.text == "💳 To‘lov")
async def payment_methods(message: types.Message):
    inline_kb = InlineKeyboardMarkup(inline_keyboard=[
        [InlineKeyboardButton(text="💳 Click orqali to‘lash", url="https://click.uz")],
        [InlineKeyboardButton(text="💳 Payme orqali to‘lash", url="https://payme.uz")]
    ])
    await message.answer("💳 To‘lov usulini tanlang:", reply_markup=inline_kb)

# 🔹 Buyurtmalarni admin panelga yuborish
@dp.callback_query(lambda c: c.data.startswith("buy_"))
async def process_order(callback_query: types.CallbackQuery):
    product_name = callback_query.data.split("_")[1]
    user = callback_query.from_user

    order_text = f"📢 Yangi buyurtma!\n👤 Mijoz: {user.full_name}\n🛍 Mahsulot: {product_name}\n📞 Aloqa: @{user.username or user.id}"
    
    # Admin panelga yuborish
    await bot.send_message(ADMIN_ID, order_text)
    
    await callback_query.message.answer("✅ Buyurtmangiz qabul qilindi!")
    await callback_query.answer()

# 🔹 Buyurtmalarim (Admin panel orqali boshqarish)
@dp.message(lambda message: message.text == "📦 Buyurtmalarim")
async def my_orders(message: types.Message):
    await message.answer("📋 Hozircha buyurtmalar tarixi mavjud emas.")

# 🔹 Aloqa bo‘limi
@dp.message(lambda message: message.text == "📞 Aloqa")
async def contact_info(message: types.Message):
    await message.answer("📞 Biz bilan bog‘lanish: +998 90 123 45 67")

# 🔹 Ma’lumot bo‘limi
@dp.message(lambda message: message.text == "ℹ️ Ma’lumot")
async def about_info(message: types.Message):
    await message.answer("🛒 Xojalíq buyumlarí - sifatlí uy-ro‘zg‘or buyumlarí arzon narxlarda!")

# 🔹 Orqaga qaytish
@dp.message(lambda message: message.text == "⬅️ Orqaga")
async def back_to_main(message: types.Message):
    await message.answer("Asosiy menyuga qaytdingiz.", reply_markup=main_menu)

# 🔹 Botni ishga tushirish
async def main():
    logging.basicConfig(level=logging.INFO)
    await dp.start_polling(bot)

if __name__ == '__main__':
    asyncio.run(main())
