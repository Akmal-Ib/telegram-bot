import telebot
import datetime
import sys
import json
from telebot import types

# =============== НАСТРОЙКИ ===============
TOKEN = "YOUR ID_TOKEN"  # вставь свой токен сюда
DATA_FILE = "users.json"   # файл для сохранения пользователей

bot = telebot.TeleBot(TOKEN)

# =============== ХРАНИЛИЩЕ ДАННЫХ ===============
def load_users():
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return {}

def save_users():
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(users, f, ensure_ascii=False, indent=4)

users = load_users()

# =============== СПРАВОЧНИК СТОЛИЦ ===============
capitals = {
    "Россия": "Москва",
    "Франция": "Париж",
    "Германия": "Берлин",
    "Япония": "Токио",
    "США": "Вашингтон",
    "Китай": "Пекин",
    "Индия": "Нью-Дели",
    "Бразилия": "Бразилиа",
    "Канада": "Оттава",
    "Италия": "Рим",
    "Великобритания": "Лондон",
    "Австралия": "Канберра",
    "Испания": "Мадрид",
    "Мексика": "Мехико",
    "Турция": "Анкара",
    "Аргентина": "Буэнос-Айрес",
    "Южная Корея": "Сеул",
    "Саудовская Аравия": "Эр-Рияд",
    "Швеция": "Стокгольм",
    "Норвегия": "Осло",
    "Финляндия": "Хельсинки",
    "Польша": "Варшава",
    "Греция": "Афины",
    "Египет": "Каир",
    "ЮАР": "Претория",
    "Нидерланды": "Амстердам",
    "Бельгия": "Брюссель",
    "Швейцария": "Берн",
    "Австрия": "Вена",
    "Чехия": "Прага",
    "Португалия": "Лиссабон",
    "Дания": "Копенгаген",
    "Ирландия": "Дублин",
    "Новая Зеландия": "Веллингтон",
    "Израиль": "Иерусалим",
    "Таиланд": "Бангкок",
    "Венгрия": "Будапешт",
    "Румыния": "Бухарест",
    "Сингапур": "Сингапур",
    "Индонезия": "Джакарта",
    "Вьетнам": "Ханой",
    "Филиппины": "Манила",
    "Малайзия": "Куала-Лумпур",
    "Пакистан": "Исламабад",
    "Бангладеш": "Дакка",
    "Украина": "Киев",
    "Беларусь": "Минск",
    "Казахстан": "Астана",
    "Узбекистан": "Ташкент",
    "Армения": "Ереван",
    "Грузия": "Тбилиси",
    "Азербайджан": "Баку",
    "Киргизия": "Бишкек",
    "Таджикистан": "Душанбе",
    "Молдова": "Кишинёв",
    "Латвия": "Рига",
    "Литва": "Вильнюс",
    "Эстония": "Таллин",
    # ... добавьте остальные страны по необходимости ...
}

# =============== КОМАНДЫ ===============
@bot.message_handler(commands=['start'])
def start_message(message):
    user_id = str(message.from_user.id)
    users[user_id] = {
        "first_name": message.from_user.first_name,
        "last_name": message.from_user.last_name,
        "username": message.from_user.username,
        "preference": None
        
    }
    save_users()

    missing = []
    if not message.from_user.first_name:
        missing.append("Имя")
    if not message.from_user.last_name:
        missing.append("Фамилия")
    if not message.from_user.username:
        missing.append("Username")
    tips = ""
    if missing:
        tips = "\n❗ Не заполнены поля: " + ", ".join(missing)

    bot.send_message(
        message.chat.id,
        f"Привет, {message.from_user.first_name or 'Пользователь'}!\n"
        f"Ты зарегистрирован ✅\nДля команд напишите /help\n\n"
        f"Твои данные:\n"
        f"ID: {user_id}\n"
        f"Имя: {message.from_user.first_name or 'нет'}\n"
        f"Фамилия: {message.from_user.last_name or 'нет'}\n"
        f"Username: @{message.from_user.username if message.from_user.username else 'нет'}"
        f"{tips}"
    )

@bot.message_handler(commands=['help'])
def help_message(message):
    bot.send_message(
        message.chat.id,
        "ℹ️ Доступные команды:\n"
        "/start — регистрация\n"
        "/help — список команд\n"
        "/time — текущее время\n"
        "/date — сегодняшняя дата\n"
        "/reverse текст — перевернуть текст\n"
        "/sum число1 число2 — сумма\n"
        "/restart — перезапуск\n"
        "/info — профиль\n"
        "/capital страна — столица страны\n"
        "/site — перейти на сайт\n"
        "/setpref — установить предпочтение\n"
        "/IT — уроки по IT"
        
    )

@bot.message_handler(commands=['time'])
def time_message(message):
    now = datetime.datetime.now().strftime("%H:%M:%S")
    bot.send_message(message.chat.id, f"⏰ Текущее время: {now}")

@bot.message_handler(commands=['date'])
def date_message(message):
    today = datetime.datetime.now().strftime("%d-%m-%Y")
    bot.send_message(message.chat.id, f"📅 Сегодня: {today}")

@bot.message_handler(commands=['reverse'])
def reverse_message(message):
    text = message.text.replace('/reverse', '').strip()
    if text:
        bot.send_message(message.chat.id, f"🔄 {text[::-1]}")
    else:
        bot.send_message(message.chat.id, "❗ Используй: /reverse текст")

@bot.message_handler(commands=['sum'])
def sum_message(message):
    try:
        parts = message.text.split()
        if len(parts) == 3:
            num1, num2 = float(parts[1]), float(parts[2])
            bot.send_message(message.chat.id, f"➕ Сумма: {num1 + num2}")
        else:
            bot.send_message(message.chat.id, "❗ Используй: /sum число1 число2")
    except ValueError:
        bot.send_message(message.chat.id, "❗ Ошибка: введи два числа")

@bot.message_handler(commands=['restart'])
def restart_message(message):
    bot.send_message(message.chat.id, "♻️ Перезапуск...")
    save_users()
    sys.exit(0)

@bot.message_handler(commands=['info'])
def info_message(message):
    user = users.get(str(message.from_user.id))
    if user:
        bot.send_message(
            message.chat.id,
            f"👤 Ваш профиль:\n"
            f"Имя: {user['first_name'] or 'нет'}\n"
            f"Фамилия: {user['last_name'] or 'нет'}\n"
            f"Username: @{user['username'] if user['username'] else 'нет'}\n"
            f"Предпочтение: {user['preference'] or 'не выбрано'}"
        )
    else:
        bot.send_message(message.chat.id, "Вы ещё не зарегистрированы. Используйте /start.")

@bot.message_handler(commands=['capital'])
def capital_command(message):
    country = message.text.replace('/capital', '').strip()
    if country:
        answer = capitals.get(country, "❌ Не знаю такую страну")
        bot.send_message(message.chat.id, f"🏙 Столица {country}: {answer}")
    else:
        bot.send_message(message.chat.id, "❗ Используй: /capital страна")

@bot.message_handler(commands=['setpref'])
def set_pref(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    markup.add("YouTube", "Wikipedia", "TikTok", "GDZ", "Whatsapp")
    bot.send_message(message.chat.id, "Выбери любимый сайт:", reply_markup=markup)

@bot.message_handler(func=lambda msg: msg.text in ["YouTube", "Wikipedia", "TikTok", "GDZ", "Whatsapp"])
def save_pref(message):
    user_id = str(message.from_user.id)
    users[user_id]['preference'] = message.text
    save_users()
    bot.send_message(message.chat.id, f"✅ Предпочтение сохранено: {message.text}", reply_markup=types.ReplyKeyboardRemove())

@bot.message_handler(commands=['site'])
def site_message(message):
    user_id = str(message.from_user.id)
    pref = users.get(user_id, {}).get('preference')
    markup = types.InlineKeyboardMarkup()

    sites = {
        "YouTube": "https://www.youtube.com/",
        "Wikipedia": "https://ru.wikipedia.org/",
        "TikTok": "https://www.tiktok.com/",
        "GDZ": "https://gdz.ru/",
        "Whatsapp": "https://web.whatsapp.com/"
    }

    if pref in sites:
        btn = types.InlineKeyboardButton(pref, url=sites[pref])
        markup.add(btn)
        bot.send_message(message.chat.id, f"🌐 Ваш сайт: {pref}", reply_markup=markup)
    else:
        for name, url in sites.items():
            markup.add(types.InlineKeyboardButton(name, url=url))
        bot.send_message(message.chat.id, "❗ У вас нет предпочтения. Выберите сайт:", reply_markup=markup)

# =============== УРОКИ IT ===============
@bot.message_handler(commands=['IT'])
def lessons_message(message):
    parts = message.text.split(maxsplit=1)
    if len(parts) == 1:
        bot.send_message(
            message.chat.id,
            "📘 Уроки по IT:\n"
            "1. Что такое компьютер и как он работает?\n"
        "2. Основы операционных систем (Windows, macOS, Linux)\n"
        "3. Введение в программирование (Python, JavaScript)\n"
        "4. Основы веб-разработки (HTML, CSS, JavaScript)\n"
        "5. Безопасность в интернете и защита данных\n"
        "6. Работа с облачными сервисами (Google Drive, Dropbox)\n"
        "7. Введение в базы данных (SQL, NoSQL)\n"
        "8. Основы сетевых технологий и интернет-протоколы\n"
        "9. Введение в мобильную разработку (Android, iOS)\n"
        "10. Современные тренды в IT (AI, Machine Learning, Blockchain)\n"
        "\nДля более подробной информации по каждому уроку, используйте соответствующие команды или задавайте вопросы!\nЧтобы узнать подробно: /IT номер (например, /IT 3)")
        
        return

    lessons = {
        "1": "💻 Компьютер — это электронное устройство, которое обрабатывает данные по заданной программе. Он состоит из аппаратного обеспечения (hardware) и программного обеспечения (software). Аппаратное обеспечение включает в себя процессор, память, устройства ввода/вывода и другие компоненты. Программное обеспечение — это набор инструкций, которые управляют работой компьютера и позволяют выполнять различные задачи.",
        "2": "🖥 Операционная система (ОС) — это программное обеспечение, которое управляет аппаратными ресурсами компьютера и предоставляет услуги для выполнения программ. Windows, macOS и Linux — это три основных типа ОС. Windows известна своей пользовательской дружелюбностью и широким распространением. macOS используется на компьютерах Apple и славится своей стабильностью и дизайном. Linux — это открытая ОС, которая популярна среди разработчиков и серверов благодаря своей гибкости и безопасности.",
        "3": "🐍 Программирование — это процесс создания программ, которые выполняют определённые задачи. Python — это высокоуровневый язык программирования, известный своей простотой и читаемостью, что делает его отличным выбором для начинающих. JavaScript — это язык программирования, который используется в основном для создания интерактивных элементов на веб-страницах. Он позволяет создавать динамические веб-приложения и широко используется в веб-разработке.",
        "4": "🌍 Веб-разработка включает в себя создание и поддержание веб-сайтов. HTML (HyperText Markup Language) используется для создания структуры веб-страниц. CSS (Cascading Style Sheets) отвечает за оформление и внешний вид страниц, включая цвета, шрифты и макеты. JavaScript добавляет интерактивность и динамическое поведение на веб-страницы, позволяя создавать сложные веб-приложения.",
        "5": "🔒 Безопасность в интернете включает в себя меры и практики, направленные на защиту данных и личной информации от несанкционированного доступа, кражи или повреждения. Важные аспекты безопасности включают использование сильных паролей, регулярное обновление программного обеспечения, использование антивирусных программ, а также осторожность при открытии ссылок и вложений в электронных письмах. Также важно быть осведомлённым о фишинговых атаках и других методах социальной инженерии.",
        "6": "☁️ Облачные сервисы позволяют хранить данные и использовать приложения через интернет, а не на локальном устройстве. Google Drive и Dropbox — это популярные облачные хранилища, которые позволяют пользователям сохранять файлы, делиться ими и совместно работать над документами в реальном времени. Облачные сервисы обеспечивают доступ к данным с любого устройства, подключённого к интернету, и предлагают функции резервного копирования и синхронизации.",
        "7": "🗄  Базы данных — это организованные коллекции данных, которые позволяют эффективно хранить, управлять и извлекать информацию. SQL (Structured Query Language) — это язык запросов, используемый для взаимодействия с реляционными базами данных, такими как MySQL и PostgreSQL. NoSQL базы данных, такие как MongoDB и Cassandra, предназначены для хранения неструктурированных данных и обеспечивают гибкость и масштабируемость для современных приложений.",
        "8": "🌐 Сетевые технологии обеспечивают обмен данными между компьютерами и другими устройствами. Интернет-протоколы, такие как TCP/IP, определяют правила и стандарты для передачи данных по сети. Основные компоненты сетевых технологий включают маршрутизаторы, коммутаторы и точки доступа, которые обеспечивают подключение устройств к сети и управление трафиком. Понимание основ сетевых технологий важно для обеспечения безопасности и эффективности работы в интернете.",
        "9": "📱  Мобильная разработка включает создание приложений для мобильных устройств, таких как смартфоны и планшеты. Android — это операционная система от Google, которая использует язык программирования Java или Kotlin для разработки приложений. iOS — это операционная система от Apple, которая использует Swift или Objective-C для создания приложений. Мобильные приложения могут быть нативными (созданными специально для одной платформы) или кроссплатформенными (работающими на нескольких платформах).",
        "10": "🤖 Современные тренды в IT включают искусственный интеллект (AI), машинное обучение (Machine Learning) и блокчейн. AI позволяет создавать системы, которые могут выполнять задачи, требующие человеческого интеллекта, такие как распознавание речи и изображений. Машинное обучение — это подмножество AI, которое использует алгоритмы для анализа данных и обучения на их основе. Блокчейн — это распределённая база данных, которая обеспечивает безопасность и прозрачность транзакций, что делает её популярной в криптовалютах и других областях."
    }

    answer = lessons.get(parts[1], "❌ Неверный номер. Используй /IT 1–10")
    bot.send_message(message.chat.id, answer)

# =============== ЭХО (только для текста без команд) ===============
@bot.message_handler(func=lambda msg: not msg.text.startswith('/'))
def echo_all(message):
    bot.send_message(message.chat.id, message.text)

# =============== ЗАПУСК БОТА ===============
print("🤖 Бот запущен!")
bot.polling()
