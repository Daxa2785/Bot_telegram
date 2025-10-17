import os
import telebot
import requests
import time

# 🔑 Telegram bot token
TELEGRAM_TOKEN = "7854974433:AAFLE3XwC1yugPfdqkO5oDVwsWRelbzOY_g"
bot = telebot.TeleBot(TELEGRAM_TOKEN)

# 🔑 OpenAI API key (buni o‘zingni kaliting bilan almashtir)
OPENAI_API_KEY = "sk-API_KALITINGNI_BU_YERGA_YOZ"

# 📁 Screenshot papka yo‘li
SCREENSHOT_DIR = "/sdcard/DCIM/Screenshots"

# 🧠 ChatGPT dan javob olish funksiyasi
def get_gpt_reply(prompt):
    headers = {
        "Authorization": f"Bearer {OPENAI_API_KEY}",
        "Content-Type": "application/json"
    }
    data = {
        "model": "gpt-4o-mini",  # Yoki gpt-3.5-turbo
        "messages": [{"role": "user", "content": prompt}]
    }
    try:
        response = requests.post("https://api.openai.com/v1/chat/completions", json=data, headers=headers, timeout=30)
        result = response.json()
        return result["choices"][0]["message"]["content"]
    except Exception as e:
        return f"⚠️ Xatolik: {e}"

# 🚀 /start buyrug‘i
@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, "Salom! 👋\n"
                                      "📸 Eng so‘nggi screenshot yuborish uchun /newshot deb yozing.\n"
                                      "💬 Yoki istalgan savolni yozing – men GPT orqali javob beraman.")

# 📸 Screenshot yuborish
@bot.message_handler(commands=['newshot'])
def new_screenshot(message):
    try:
        files = [os.path.join(SCREENSHOT_DIR, f) for f in os.listdir(SCREENSHOT_DIR) if os.path.isfile(os.path.join(SCREENSHOT_DIR, f))]
        if not files:
            bot.send_message(message.chat.id, "Hech qanday screenshot topilmadi 😔")
            return

        latest = max(files, key=os.path.getctime)
        with open(latest, "rb") as photo:
            bot.send_photo(message.chat.id, photo, caption="📸 Eng so‘nggi screenshot yuborildi!")
    except Exception as e:
        bot.send_message(message.chat.id, f"⚠️ Xatolik: {e}")

# 💬 ChatGPT bilan suhbat
@bot.message_handler(func=lambda message: True)
def chat_with_gpt(message):
    user_text = message.text
    bot.send_chat_action(message.chat.id, "typing")
    reply = get_gpt_reply(user_text)
    bot.send_message(message.chat.id, reply)

# 🔄 Botni ishga tushirish
print("🤖 Bot ishga tushdi...")
bot.polling(non_stop=True)
