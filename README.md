# requirements.txt
import telebot
from telebot import types

TOKEN = '8395974750:AAEvbp2ARnsJcEgJzkMosqW_LHvcOoTsDFY'
ADMIN_ID = 8337257576

bot = telebot.TeleBot(TOKEN)
message_map = {}

@bot.message_handler(commands=['start'])
def start(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("📩 ارسال پیام")
    bot.send_message(
        message.chat.id,
        "سلام 👋\nپیامت رو بفرست، مستقیم به ادمین می‌رسه.",
        reply_markup=markup
    )

@bot.message_handler(func=lambda m: m.text == "📩 ارسال پیام")
def ask(message):
    bot.send_message(message.chat.id, "✍️ پیامت رو بنویس")

@bot.message_handler(func=lambda m: True)
def user_message(message):
    if message.chat.id == ADMIN_ID:
        return

    username = message.from_user.username or "بدون_یوزرنیم"

    sent = bot.send_message(
        ADMIN_ID,
        f"📨 پیام جدید\n👤 @{username}\n\n{message.text}"
    )

    message_map[sent.message_id] = message.chat.id
    bot.send_message(message.chat.id, "✅ پیامت ارسال شد")

@bot.message_handler(func=lambda m: m.reply_to_message is not None)
def admin_reply(message):
    if message.chat.id != ADMIN_ID:
        return

    replied_id = message.reply_to_message.message_id
    if replied_id in message_map:
        user_id = message_map[replied_id]
        bot.send_message(user_id, f"✉️ پاسخ ادمین:\n{message.text}")
        bot.send_message(ADMIN_ID, "✅ جواب ارسال شد")

bot.infinity_polling()
