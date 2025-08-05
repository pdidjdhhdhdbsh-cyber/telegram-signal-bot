# telegram-signal-bot
بوت إرسال إشارات تداول CALL / PUT يدويًا على تيليجرام
import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes
import asyncio

# إعدادات البوت
TOKEN = "ضع_التوكن_هنا"
ALLOWED_CHAT_ID = 7869556026  # استبدله بـ Chat ID الخاص بك فقط

# عدد الضغطات
button_counts = {"CALL": 0, "PUT": 0}

# دالة بدء التشغيل
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_chat.id != ALLOWED_CHAT_ID:
        await update.message.reply_text("🚫 هذا البوت خاص ولا يمكن استخدامه.")
        return

    keyboard = [
        [InlineKeyboardButton("📈 CALL", callback_data="CALL"),
         InlineKeyboardButton("📉 PUT", callback_data="PUT")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text("اختر نوع الإشارة:", reply_markup=reply_markup)

# دالة التعامل مع الضغط على الأزرار
async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.message.chat.id != ALLOWED_CHAT_ID:
        await query.edit_message_text("🚫 هذا البوت خاص.")
        return

    choice = query.data
    button_counts[choice] += 1

    await query.edit_message_text(
        text=f"📊 تم اختيار: {choice}\n\n✅ CALL: {button_counts['CALL']} مرة\n❌ PUT: {button_counts['PUT']} مرة"
    )

# دالة إرسال الإشارة يدوياً
async def signal(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if update.effective_chat.id != ALLOWED_CHAT_ID:
        return

    if len(context.args) != 1 or context.args[0] not in ["CALL", "PUT"]:
        await update.message.reply_text("❗ استخدم الأمر بهذا الشكل:\n/signal CALL أو /signal PUT")
        return

    direction = context.args[0]
    keyboard = [
        [InlineKeyboardButton("📈 CALL", callback_data="CALL"),
         InlineKeyboardButton("📉 PUT", callback_data="PUT")]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text(
        f"🚨 إشارة تداول جديدة:\nالاتجاه: {direction}", reply_markup=reply_markup
    )

# تشغيل البوت
if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("signal", signal))
    app.add_handler(CallbackQueryHandler(button_handler))

    print("✅ البوت يعمل...")
    app.run_polling()
