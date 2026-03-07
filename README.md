""" README (embedded)

ð# \xf0# \xf0\x9f¤# \xf0\x9f\xa4# \xf0\x9f\xa4\x96ð# \xf0\x9f\xa4\x96\xf0# \xf0\x9f\xa4\x96\xf0\x9f# \xf0\x9f\xa4\x96\xf0\x9f\x90±# \xf0\x9f\xa4\x96\xf0\x9f\x90\xb1 Doraemon AI Bot

Telegram AI Assistant âTelegram AI Assistant \xe2Telegram AI Assistant \xe2\x80Telegram AI Assistant \xe2\x80\x93 ChatGPT-Level Intelligence with Doraemon Spirit**

[Banner and badges omitted in code file — keep them in repository as README.md images]


---

🌌 نظرة عامة

Doraemon AI Bot هو بوت ذكاء اصطناعي متقدم يعمل على Telegram، مستوحى من شخصية Doraemon، لكنه مبني على ذكاء اصطناعي حقيقي بمستوى ChatGPT.

(ملف الكود هذا يحتوي على تنفيذ أساسي لبوت تلغرام يدعم أوامر: /start, /help, /ip, /Download_videos, /بحث, /voice ويعرض كيفية الربط مع OpenAI. جميع الميزات المتقدمة المتعلقة بأدوات الاختبار الأمني مُدرجة كأوصاف/تعليمية فقط — لا يقوم الكود بأي نشاط ضار أو غير قانوني.)


---

المتطلبات (requirements.txt)

telebot (pyTelegramBotAPI), openai, python-dotenv, requests, yt-dlp, gTTS, pydub

requirements.txt (ضع في ملف منفصل قبل التشغيل)

pyTelegramBotAPI==4.12.0

openai==0.27.8

python-dotenv==1.0.0

requests==2.31.0

yt-dlp==2024.12.22

gTTS==2.4.0

pydub==0.25.1


---

ملف المثال .env.example

TELEGRAM_TOKEN=your_telegram_bot_token

OPENAI_API_KEY=your_openai_api_key

ADMIN_ID=8196476936


---

"""

---------------------------------------------

Doraemon AI Bot - main.py

---------------------------------------------

ملاحظة مهمة:

- لا تضع مفاتيح API الحقيقية داخل الكود. استخدم متغيرات بيئية (.env).

- هذا الكود يعطي تنفيذًا آمنًا وتعليميًا فقط. الميزات المتعلقة بالاختبار الأمني تظهر كأدوات تعليمية/شرحية.

- قبل تشغيل قسم تحميل الفيديوهات تأكد من الامتثال لسياسات حقوق النشر.

import os import logging import tempfile import traceback from functools import wraps from dotenv import load_dotenv import telebot from telebot import types import requests import openai from gtts import gTTS from pydub import AudioSegment import yt_dlp

load .env

load_dotenv() TELEGRAM_TOKEN = os.getenv('TELEGRAM_TOKEN') OPENAI_API_KEY = os.getenv('OPENAI_API_KEY') ADMIN_ID = int(os.getenv('ADMIN_ID') or 0)

if not TELEGRAM_TOKEN: raise RuntimeError('TELEGRAM_TOKEN not set in environment') if not OPENAI_API_KEY: raise RuntimeError('OPENAI_API_KEY not set in environment')

openai.api_key = OPENAI_API_KEY bot = telebot.TeleBot(TELEGRAM_TOKEN, parse_mode='HTML')

logging.basicConfig(level=logging.INFO) logger = logging.getLogger(name)

-----------------------------

Helpers

-----------------------------

def restricted(func): """Decorator to restrict commands to admin only if ADMIN_ID is set (non-zero).""" @wraps(func) def wrapper(message, *args, **kwargs): if ADMIN_ID and message.from_user.id != ADMIN_ID: bot.reply_to(message, "⚠️ هذا الأمر محجوز لمالك البوت.") return return func(message, *args, **kwargs) return wrapper

def ask_openai_system(prompt, system="You are a helpful assistant."): """Call OpenAI ChatCompletion (gpt-like).""" try: resp = openai.ChatCompletion.create( model="gpt-4o-mini", # placeholder model name — replace if necessary messages=[ {"role": "system", "content": system}, {"role": "user", "content": prompt} ], max_tokens=800 ) return resp['choices'][0]['message']['content'].strip() except Exception as e: logger.exception('OpenAI call failed') return f"خطأ في الاتصال بـ OpenAI: {e}"

-----------------------------

Command Handlers

-----------------------------

@bot.message_handler(commands=['start']) def cmd_start(message: types.Message): user = message.from_user txt = ( f"مرحباً {user.first_name}! 👋\n\n" "أنا Doraemon AI Bot — مساعدك الذكي داخل Telegram. اكتب /help لعرض الأوامر." ) bot.send_message(message.chat.id, txt)

@bot.message_handler(commands=['help']) def cmd_help(message: types.Message): help_text = ( "قائمة الأوامر المتاحة:\n" "/start - بدء المحادثة\n" "/help - عرض هذه المساعدة\n" "/ask <سؤال> - اسأل الـ AI" + " (مثال: /ask اشرح الخوارزمية)\n" "/ip <ip_or_domain> - معلومات IP تعليمية\n" "/Download_videos <url> - تحميل فيديو (تأكد من الحقوق)\n" "/بحث <username> - OSINT تعليمي (معلومات عامة فقط)\n" "/voice <نص> - تحويل نص إلى صوت (mp3)\n" ) bot.send_message(message.chat.id, help_text)

@bot.message_handler(commands=['ask']) def cmd_ask(message: types.Message): prompt = message.text.partition(' ')[2].strip() if not prompt: bot.reply_to(message, 'فضلاً أرسل سؤالك بعد الأمر، مثال: /ask كيف أكتب دالة بايثون؟') return bot.send_chat_action(message.chat.id, 'typing') reply = ask_openai_system(prompt, system="You are Doraemon AI, friendly and concise. Reply in Arabic if the user used Arabic.") bot.reply_to(message, reply)

@bot.message_handler(commands=['ip']) def cmd_ip(message: types.Message): target = message.text.partition(' ')[2].strip() if not target: bot.reply_to(message, 'أرسل IP أو نطاق بعد الأمر، مثال: /ip 8.8.8.8') return bot.send_chat_action(message.chat.id, 'typing') try: # استخدم خدمة ip-api (مجانية) لمعلومات أساسية r = requests.get(f'http://ip-api.com/json/{target}?fields=status,country,regionName,city,isp,org,query') data = r.json() if data.get('status') != 'success': bot.reply_to(message, f'لم أجد معلومات عن: {target}') return msg = ( f"📡 معلومات عن {data.get('query')}\n" f"البلد: {data.get('country')}\n" f"المنطقة: {data.get('regionName')}\n" f"المدينة: {data.get('city')}\n" f"موفر الخدمة: {data.get('isp')}\n" f"المنظمة: {data.get('org')}\n" ) bot.reply_to(message, msg) except Exception as e: logger.exception('ip lookup failed') bot.reply_to(message, f'حدث خطأ أثناء جلب معلومات IP: {e}')

@bot.message_handler(commands=['Download_videos']) def cmd_download_videos(message: types.Message): url = message.text.partition(' ')[2].strip() if not url: bot.reply_to(message, 'أرسل رابط الفيديو بعد الأمر') return # تحذير حقوق النشر bot.send_message(message.chat.id, '⚠️ تأكد من أنك تمتلك الحق في تنزيل هذا المحتوى. سيتم محاولة تنزيل نسخة تعليمية فقط.') bot.send_chat_action(message.chat.id, 'upload_video') try: with tempfile.TemporaryDirectory() as tmpdir: ydl_opts = { 'format': 'best[ext=mp4]/best', 'outtmpl': os.path.join(tmpdir, '%(title)s.%(ext)s'), 'noplaylist': True, 'quiet': True, } with yt_dlp.YoutubeDL(ydl_opts) as ydl: info = ydl.extract_info(url, download=True) filename = ydl.prepare_filename(info) # send as file (size permitting) filesize = os.path.getsize(filename) max_send = 50 * 1024 * 1024  # 50 MB حدود Telegram للأرسل كملف - قد تختلف حسب البوت if filesize > max_send: bot.send_message(message.chat.id, 'حجم الملف كبير جدًا، لا يمكن إرساله عبر البوت. ارفع الملف إلى خدمة سحابية.') return with open(filename, 'rb') as f: bot.send_video(message.chat.id, f) except Exception as e: logger.exception('download failed') bot.reply_to(message, f'فشل تنزيل الفيديو: {e}')

@bot.message_handler(commands=['بحث']) def cmd_search(message: types.Message): q = message.text.partition(' ')[2].strip() if not q: bot.reply_to(message, 'اكتب اسم المستخدم أو المعطى بعد الأمر للمُحاولة البحث OSINT تعليميًا.') return bot.send_chat_action(message.chat.id, 'typing') # تنفيذ تعليمات OSINT عامة — لا نجمع بيانات شخصية حساسة prompt = ( f"أعطِ خطوات OSINT عامة وآمنة للبحث عن '{q}' عبر الويب ووسائل التواصل الاجتماعي." " اذكر أدوات مشهورة (تعليمي) ولا تقم بأي نشاط اختراقي.") reply = ask_openai_system(prompt, system="You are an OSINT-ethics-aware assistant. Provide high-level, lawful guidance.") bot.reply_to(message, reply)

@bot.message_handler(commands=['voice']) def cmd_voice(message: types.Message): text = message.text.partition(' ')[2].strip() if not text: bot.reply_to(message, 'أرسل النص بعد الأمر ليتم تحويله إلى صوت، مثال: /voice مرحبًا') return bot.send_chat_action(message.chat.id, 'record_audio') try: tts = gTTS(text=text, lang='ar') with tempfile.NamedTemporaryFile(suffix='.mp3', delete=False) as tf: tmp_mp3 = tf.name tts.save(tmp_mp3) # optional: convert to ogg if desired bot.send_audio(message.chat.id, open(tmp_mp3, 'rb')) os.remove(tmp_mp3) except Exception as e: logger.exception('tts failed') bot.reply_to(message, f'فشل تحويل النص إلى صوت: {e}')

-----------------------------

Admin / Advanced (educational only)

-----------------------------

@bot.message_handler(commands=['dantest']) @restricted def cmd_dan_test(message: types.Message): # لا ننفذ أوامر ضارة — نعرض أمثلة تعليمية فقط txt = ( "DAN Mode (تعليمي فقط):\n" "• يمكنني شرح تقنيات اختبار الاختراق الأخلاقي مثل Nmap, SQLi, XSS على المستوى النظري.\n" "• لا أنفذ هجمات ولا أزود أوامر جاهزة للإستغلال.\n" "اطلب /daninfo <topic> لشرح نظري." ) bot.reply_to(message, txt)

@bot.message_handler(commands=['daninfo']) @restricted def cmd_dan_info(message: types.Message): topic = message.text.partition(' ')[2].strip() or 'nmap' prompt = f" اشرح بشكل تعليمي وأخلاقي ما هو {topic} وكيفية استخدامه بشكل قانوني لأغراض اختبار الاختراق الأخلاقي " reply = ask_openai_system(prompt, system="You are an expert in cybersecurity who only provides lawful, ethical guidance.") bot.reply_to(message, reply)

-----------------------------

Error handling, fallback

-----------------------------

@bot.message_handler(func=lambda m: True, content_types=['text']) def handle_all_text(message: types.Message): # محادثة عامة: نستخدم OpenAI لتوليد رد text = message.text.strip() if len(text) < 4: bot.reply_to(message, "اكتب شيئًا أطول لأستطيع المساعدة.") return bot.send_chat_action(message.chat.id, 'typing') try: reply = ask_openai_system(text, system="You are Doraemon AI: helpful, concise, technical when needed. Reply in user's language.") bot.reply_to(message, reply) except Exception as e: logger.exception('chat fallback failed') bot.reply_to(message, 'حدث خطأ أثناء معالجة الرسالة.')

-----------------------------

Run

-----------------------------

if name == 'main': print('Starting Doraemon AI Bot...') try: bot.infinity_polling(timeout=60, long_polling_timeout = 5) except KeyboardInterrupt: print('Stopped by user') except Exception: traceback.print_exc() print('Bot stopped due to error')

---------------------------------------------

Notes & Next steps (suggestions for repo):

- ضع README.md (النص الكامل الذي أرسلته) في جذر المشروع.

- أضف صور banner.png, chat.png, code.png في مجلد /assets أو root.

- أضف ملف LICENSE (مثلاً MIT) إذا أردت ترخيصًا مفتوحًا.

- لإنشاء نسخة قابلة للنشر على Heroku/VPS: أنشئ Procfile أو systemd unit.

- أضف اختبارات وحدات وخط أنابيب CI (GitHub Actions) لنشر آمن.

نهاية الملف
