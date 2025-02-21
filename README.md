# ALI-BOT


import logging
import json
import os
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, CallbackQueryHandler, CallbackContext
from dotenv import load_dotenv

# تحميل المتغيرات البيئية
load_dotenv()
TOKEN = os.getenv("BOT_TOKEN")

# إعداد السجلات
logging.basicConfig(format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO)
logger = logging.getLogger(__name__)

# تحميل الأرصدة من ملف JSON
def load_balances():
    try:
        with open("balance.json", "r", encoding="utf-8") as file:
            data = json.load(file)
        return data.get("balances", {})
    except (FileNotFoundError, json.JSONDecodeError):
        return {}

# تحديث الرصيد وحفظه في ملف JSON
def update_balance(user_id, amount):
    balances = load_balances()
    balances[str(user_id)] = balances.get(str(user_id), 0) + amount
    with open("balance.json", "w", encoding="utf-8") as file:
        json.dump({"balances": balances}, file, indent=4)

# عرض الرصيد الحالي للمستخدم
async def check_balance(update: Update, context: CallbackContext) -> None:
    user_id = str(update.message.from_user.id)
    balances = load_balances()
    balance = balances.get(user_id, 0)
    await update.message.reply_text(f"\U0001F4B0 رصيدك الحالي: {balance} USDT")

# وظيفة بدء البوت
async def start(update: Update, context: CallbackContext) -> None:
    keyboard = [
        [InlineKeyboardButton("\U0001F4CC قائمة الحسابات", callback_data="list_accounts")],
        [InlineKeyboardButton("\U0001F6D2 شراء حساب", callback_data="buy_account")],
        [InlineKeyboardButton("\U0001F4B0 معرفة الرصيد", callback_data="check_balance")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("\U0001F4BB مرحبًا بك في متجر الحسابات! اختر ما تريد:", reply_markup=reply_markup)

# التعامل مع الأزرار
async def button_handler(update: Update, context: CallbackContext) -> None:
    query = update.callback_query
    user_id = str(query.from_user.id)
    await query.answer()
    
    if query.data == "check_balance":
        balances = load_balances()
        balance = balances.get(user_id, 0)
        await query.edit_message_text(f"\U0001F4B0 رصيدك الحالي: {balance} USDT")

# تشغيل البوت
async def main():
    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("balance", check_balance))
    app.add_handler(CallbackQueryHandler(button_handler))
    
    print("✅ Bot is running...")
    await app.run_polling()

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())