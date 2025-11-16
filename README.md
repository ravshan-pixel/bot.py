import requests
import json

BOT_TOKEN = "8284100332:AAF_GlynuU3199KMEaorN7O29wWbwsSh-oA"
BASE_URL = f"https://api.telegram.org/bot{BOT_TOKEN}"

# Foydalanuvchilar til tanlovini saqlash
user_language = {}

# Klaviatura yaratish
def get_main_keyboard():
    keyboard = {
        "keyboard": [
            [
                {"text": "🇺🇿 Uzbek -> Русский 🇷🇺"},
                {"text": "🇷🇺 Русский -> Uzbek 🇺🇿"}
            ],
            [
                {"text": "ℹ️ Yordam"},
                {"text": "🔄 Tilni almashtirish"}
            ]
        ],
        "resize_keyboard": True
    }
    return keyboard

# Google Translate API orqali tarjima qilish
def translate_text(text, target_lang):
    try:
        url = "https://translate.googleapis.com/translate_a/single"
        params = {
            'client': 'gtx',
            'sl': 'auto',
            'tl': target_lang,
            'dt': 't',
            'q': text
        }
        response = requests.get(url, params=params)
        result = response.json()
        return result[0][0][0]
    except:
        return None

# Xabar yuborish
def send_message(chat_id, text, reply_markup=None):
    url = f"{BASE_URL}/sendMessage"
    data = {
        "chat_id": chat_id,
        "text": text
    }
    if reply_markup:
        data["reply_markup"] = reply_markup
    requests.post(url, json=data)

# Yangi xabarlarni olish
def get_updates(offset=None):
    url = f"{BASE_URL}/getUpdates"
    params = {"timeout": 100, "offset": offset}
    response = requests.get(url, params=params)
    return response.json()

# Xabarlarni qayta ishlash
def handle_updates():
    last_update_id = None
    while True:
        updates = get_updates(last_update_id)
        if "result" in updates:
            for update in updates["result"]:
                last_update_id = update["update_id"] + 1
                if "message" in update and "text" in update["message"]:
                    message = update["message"]
                    chat_id = message["chat"]["id"]
                    user_id = message["from"]["id"]
                    text = message["text"]
                    
                    # Start komandasi
                    if text == "/start":
                        user_language[user_id] = None
                        send_message(
                            chat_id,
                            "Assalomu alaykum! Kuchli tarjimon botga xush kelibsiz!\n\n"
                            "Здравствуйте! Добро пожаловать в мощный переводчик!\n\n"
                            "Istalgan gap yoki matnni yuboring, men uni tarjima qilaman.",
                            get_main_keyboard()
                        )
                    
                    # Til tanlash
                    elif text == "🇺🇿 Uzbek -> Русский 🇷🇺":
                        user_language[user_id] = "uz-ru"
                        send_message(
                            chat_id,
                            "✅ Uzbek tilidan Rus tiliga tarjima rejimi aktivlashdi!\n"
                            "O'zbekcha istalgan matn yuboring, men rus tiliga tarjima qilaman."
                        )
                    
                    elif text == "🇷🇺 Русский -> Uzbek 🇺🇿":
                        user_language[user_id] = "ru-uz"
                        send_message(
                            chat_id,
                            "✅ Режим перевода с русского на узбекский активирован!\n"
                            "Отправьте любой текст на русском, я переведу на узбекский."
                        )
                    
                    # Yordam
                    elif text == "ℹ️ Yordam":
                        help_text = (
                            "🤖 Kuchli Tarjimon Bot\n\n"
                            "Qo'llanma:\n"
                            "1. Avval tarjima yo'nalishini tanlang\n"
                            "2. Keyin istalgan gap yoki matn yuboring\n"
                            "3. Bot to'liq matnni avtomatik tarjima qiladi\n\n"
                            "Инструкция:\n"
                            "1. Сначала выберите направление перевода\n"
                            "2. Затем отправьте любой текст или предложение\n"
                            "3. Бот автоматически переведет полный текст"
                        )
                        send_message(chat_id, help_text)
                    
                    # Tilni almashtirish
                    elif text == "🔄 Tilni almashtirish":
                        user_language[user_id] = None
                        send_message(
                            chat_id,
                            "Tarjima yo'nalishi o'chirildi. Iltimos, yangi yo'nalishni tanlang.\n"
                            "Направление перевода сброшено. Пожалуйста, выберите новое направление.",
                            get_main_keyboard()
                        )
                    
                    # Tarjima qilish
                    else:
                        if user_id not in user_language or user_language[user_id] is None:
                            send_message(
                                chat_id,
                                "❌ Iltimos, avval tarjima yo'nalishini tanlang!\n"
                                "❌ Пожалуйста, сначала выберите направление перевода!",
                                get_main_keyboard()
                            )
                        else:
                            direction = user_language[user_id]
                            if direction == "uz-ru":
                                translated = translate_text(text, "ru")
                                if translated:
                                    send_message(
                                        chat_id,
                                        f"🔹 Asl matn: {text}\n\n🔹 Перевод: {translated}"
                                    )
                                else:
                                    send_message(chat_id, "❌ Tarjima qilishda xatolik yuz berdi.")
                            elif direction == "ru-uz":
                                translated = translate_text(text, "uz")
                                if translated:
                                    send_message(
                                        chat_id,
                                        f"🔹 Исходный текст: {text}\n\n🔹 Tarjima: {translated}"
                                    )
                                else:
                                    send_message(chat_id, "❌ Tarjima qilishda xatolik yuz berdi.")

if __name__ == "__main__":
    print("🤖 Kuchli tarjimon bot muvaffaqiyatli ishga tushdi!")
    handle_updates()
