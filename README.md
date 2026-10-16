import asyncio
import json
import os
import uuid
import aiohttp

from rubka import Robot


# ==========================================
# تنظیمات اصلی ربات
# ==========================================

import os

TOKEN = os.getenv("CGJIGG0QSITOIGOPVOPXAKTGTNGKZBYFKEJWJVBRLFJNTHKOCAJQERTGHBVQPMHP")
ADMIN_ID = os.getenv("u0JdgSA07a61484032ddf92f60ca7770")
ADMIN_USERNAME = "@PV_ZHNERAL"

DATA_FILE = "general_shop_data.json"


# ==========================================
# اتصال و اصلاح DNS
# ==========================================

bot = Robot(token=TOKEN)


async def fixed_get_session():
    session = getattr(bot, "_aiohttp_session", None)

    if session is None or session.closed:
        connector = aiohttp.TCPConnector(
            limit=100,
            ssl=False,
            resolver=aiohttp.resolver.ThreadedResolver()
        )

        timeout = aiohttp.ClientTimeout(
            total=getattr(bot, "timeout", 30)
        )

        bot._aiohttp_session = aiohttp.ClientSession(
            connector=connector,
            timeout=timeout
        )

    return bot._aiohttp_session


bot._get_session = fixed_get_session


# ==========================================
# ذخیره و خواندن اطلاعات
# ==========================================

def load_data():
    if not os.path.exists(DATA_FILE):
        return {
            "users": {},
            "inventory": [],
            "pending_sales": []
        }

    try:
        with open(DATA_FILE, "r", encoding="utf-8") as file:
            data = json.load(file)

        data.setdefault("users", {})
        data.setdefault("inventory", [])
        data.setdefault("pending_sales", [])

        return data

    except (json.JSONDecodeError, OSError):
        return {
            "users": {},
            "inventory": [],
            "pending_sales": []
        }


data = load_data()
states = {}


def save_data():
    temporary_file = DATA_FILE + ".tmp"

    with open(temporary_file, "w", encoding="utf-8") as file:
        json.dump(
            data,
            file,
            ensure_ascii=False,
            indent=2
        )

    os.replace(temporary_file, DATA_FILE)


# ==========================================
# متن‌های فارسی و انگلیسی
# ==========================================

TEXTS = {
    "fa": {
        "language": "زبان خود را انتخاب کنید ❣",
        "welcome": (
            "سلام دنبال کننده ژنرال ❣\n"
            "به ربات شاپ ژنرال خوش آمدید☺️"
        ),
        "sell": (
            "⭕️ لطفا موجودی خود را ارسال کنید "
            "[ همراه لینک چنل یا گپ ]"
        ),
        "sent": "✅ موجودی شما برای ادمین ارسال شد.",
        "accepted": (
            "موجودی شما به لیست اضافه شد ✅\n"
            "جهت اطلاعات بیشتر با ادمین در ارتباط باشید 👇\n"
            "@PV_ZHNERAL"
        ),
        "rejected": (
            "سفارش شما رد شد ⭕️\n"
            "جهت اطلاعات بیشتر با ادمین در ارتباط باشید 👇\n"
            "@PV_ZHNERAL"
        ),
        "wait": "لطفا ایدی روبیکایی خود را ارسال کنید ⭕️",
        "id_sent": "✅ آیدی شما برای ادمین ارسال شد.",
        "admin_add": "لطفا موجودی خود را ارسال کنید تا در ربات گذاشته شود ✅",
        "admin_remove": "لطفا متن موجودی یا کد موجودی را ارسال کنید تا لغو شود ⭕️",
        "empty": "⭕️ در حال حاضر موجودی‌ای وجود ندارد.",
        "back": "برگشت 🔙",
        "not_found": "⭕️ موجودی موردنظر پیدا نشد.",
        "error": "❌ هنگام انجام عملیات خطایی رخ داد.",
        "added": "✅ موجودی با موفقیت اضافه شد.",
        "removed": "✅ موجودی با موفقیت حذف شد.",
        "admin": "پنل مدیریت ژنرال",
        "choose": "لطفا یکی از گزینه‌های زیر را انتخاب کنید.",
        "inventory": "📦 فهرست موجودی‌های ژنرال:",
        "code": "کد موجودی",
        "new_sale": "🔔 درخواست فروش جدید",
        "accept": "تأیید ✅",
        "reject": "رد کردن ⭕️",
        "wait_button": "دریافت آیدی 🆔",
        "already_done": "این درخواست قبلاً بررسی شده است.",
        "only_admin": "⛔️ این دکمه فقط برای ادمین است.",
        "myid": "شناسه کاربر",
        "chatid": "شناسه گفتگو",
        "no_content": "پیام خالی است؛ لطفاً متن یا فایل بفرستید."
    },

    "en": {
        "language": "Choose your language ❣",
        "welcome": (
            "Hello General follower ❣\n"
            "Welcome to General Shop Bot ☺️"
        ),
        "sell": (
            "⭕️ Please send your inventory "
            "[ with channel or group link ]"
        ),
        "sent": "✅ Your inventory has been sent to the admin.",
        "accepted": (
            "Your inventory has been added to the list ✅\n"
            "For more information contact admin 👇\n"
            "@PV_ZHNERAL"
        ),
        "rejected": (
            "Your order was rejected ⭕️\n"
            "For more information contact admin 👇\n"
            "@PV_ZHNERAL"
        ),
        "wait": "Please send your Rubika ID ⭕️",
        "id_sent": "✅ Your ID has been sent to the admin.",
        "admin_add": "Please send the inventory to add it to the bot ✅",
        "admin_remove": "Please send the inventory text or inventory code to delete it ⭕️",
        "empty": "⭕️ There is currently no inventory.",
        "back": "Back 🔙",
        "not_found": "⭕️ Inventory not found.",
        "error": "❌ An error occurred while processing the request.",
        "added": "✅ Inventory added successfully.",
        "removed": "✅ Inventory deleted successfully.",
        "admin": "General Admin Panel",
        "choose": "Please select one of the options below.",
        "inventory": "📦 General Inventory List:",
        "code": "Inventory code",
        "new_sale": "🔔 New selling request",
        "accept": "Accept ✅",
        "reject": "Reject ⭕️",
        "wait_button": "Request ID 🆔",
        "already_done": "This request has already been processed.",
        "only_admin": "⛔️ This button is for the admin only.",
        "myid": "User ID",
        "chatid": "Chat ID",
        "no_content": "Empty message; please send text or a file."
    }
}


# ==========================================
# ابزارهای کمکی
# ==========================================

def get_user_id(message):
    return str(
        getattr(message, "sender_id", None)
        or getattr(message, "chat_id", "")
    )


def get_chat_id(message):
    return str(
        getattr(message, "chat_id", None)
        or getattr(message, "sender_id", "")
    )


def is_admin(message):
    admin_id = str(ADMIN_ID).strip()

    return admin_id in (
        get_user_id(message),
        get_chat_id(message)
    )


def get_language(user_id):
    user = data["users"].get(str(user_id), {})
    return user.get("language", "fa")


def tr(user_id, key):
    language = get_language(user_id)
    return TEXTS.get(language, TEXTS["fa"]).get(
        key,
        TEXTS["fa"].get(key, key)
    )


def create_code():
    existing_codes = {
        item.get("code")
        for item in data["inventory"]
    }

    while True:
        code = "GN-" + uuid.uuid4().hex[:6].upper()

        if code not in existing_codes:
            return code


def get_file_id(message):
    file_object = getattr(message, "file", None)

    if file_object is None:
        return None

    return getattr(file_object, "file_id", None)


def get_media_type(message):
    file_id = get_file_id(message)

    if not file_id:
        return "text"

    if getattr(message, "is_photo", False):
        return "photo"

    if getattr(message, "is_video", False):
        return "video"

    if getattr(message, "is_document", False):
        return "document"

    filename = str(
        getattr(message.file, "file_name", "") or ""
    ).lower()

    if filename.endswith((".jpg", ".jpeg", ".png", ".webp")):
        return "photo"

    if filename.endswith((".mp4", ".mov", ".mkv", ".webm")):
        return "video"

    return "document"


def get_message_content(message):
    return str(getattr(message, "text", "") or "").strip()


def create_inventory(message, user_id):
    content = get_message_content(message)
    media_type = get_media_type(message)
    file_id = get_file_id(message)

    if media_type == "text" and not content:
        return None

    if media_type != "text" and not file_id:
        return None

    return {
        "code": create_code(),
        "type": media_type,
        "caption": content,
        "file_id": file_id,
        "user_id": str(user_id)
    }


async def send_media(chat_id, item):
    media_type = item.get("type", "text")
    content = item.get("caption", "")
    file_id = item.get("file_id")

    if media_type == "text" or not file_id:
        text = content or " "
        return await bot.send_message(
            chat_id=str(chat_id),
            text=text
        )

    caption = (
        f"{content}\n\n"
        if content else ""
    ) + f"کد موجودی: {item.get('code', '---')}"

    if media_type == "photo":
        return await bot.send_photo(
            chat_id=str(chat_id),
            file_id=file_id,
            caption=caption
        )

    if media_type == "video":
        return await bot.send_video(
            chat_id=str(chat_id),
            file_id=file_id,
            text=caption
        )

    return await bot.send_document(
        chat_id=str(chat_id),
        file_id=file_id,
        text=caption
    )


# ==========================================
# صفحه‌کلیدها
# ==========================================

def chat_keyboard(rows):
    return {
        "rows": [
            {
                "buttons": [
                    {
                        "id": text,
                        "type": "Simple",
                        "button_text": text
                    }
                    for text in row
                ]
            }
            for row in rows
        ],
        "resize_keyboard": True
    }


def inline_keyboard(rows):
    return {
        "rows": [
            {
                "buttons": [
                    {
                        "id": button_id,
                        "type": "Simple",
                        "button_text": text
                    }
                    for button_id, text in row
                ]
            }
            for row in rows
        ]
    }


def user_keyboard(user_id):
    if get_language(user_id) == "en":
        rows = [
            ["Inventory List"],
            ["Selling inventory to admin"],
            ["Back 🔙"]
        ]
    else:
        rows = [
            ["لیست موجودی ها┊𝐈𝐧𝐯𝐞𝐧𝐭𝐨𝐫𝐲 𝐋𝐢𝐬𝐭"],
            ["𝐒𝐞𝐥𝐥𝐢𝐧𝐠 𝐢𝐧𝐯𝐞𝐧𝐭𝐨𝐫𝐲 𝐭𝐨 𝐭𝐡𝐞 𝐚𝐝𝐦𝐢𝐧 ┊فروش موجودی به ادمین"],
            ["برگشت 🔙"]
        ]

    return chat_keyboard(rows)


def admin_keyboard():
    return chat_keyboard([
        ["افزودن موجودی ✅"],
        ["لغو کردن یکی از موجودی ها ⭕️"]
    ])


async def show_home(chat_id, user_id):
    if is_admin_id(user_id):
        await bot.send_message(
            chat_id=str(chat_id),
            text=tr(user_id, "admin") + "\n" + tr(user_id, "choose"),
            chat_keypad=admin_keyboard()
        )
    else:
        await bot.send_message(
            chat_id=str(chat_id),
            text=tr(user_id, "welcome"),
            chat_keypad=user_keyboard(user_id)
        )


def is_admin_id(user_id):
    return str(user_id).strip() == str(ADMIN_ID).strip()


# ==========================================
# نمایش فهرست موجودی‌ها
# ==========================================

async def show_inventory(chat_id, user_id):
    if not data["inventory"]:
        await bot.send_message(
            chat_id=str(chat_id),
            text=tr(user_id, "empty")
        )
        return

    await bot.send_message(
        chat_id=str(chat_id),
        text=tr(user_id, "inventory")
    )

    for item in data["inventory"]:
        try:
            code = item.get("code", "---")
            content = item.get("caption", "")

            if item.get("type") == "text":
                await bot.send_message(
                    chat_id=str(chat_id),
                    text=(
                        f"{content}\n\n"
                        f"{tr(user_id, 'code')}: {code}"
                    )
                )
            else:
                await send_media(chat_id, item)

        except Exception as error:
            print("Inventory send error:", error)

            try:
                await bot.send_message(
                    chat_id=str(chat_id),
                    text=f"کد موجودی: {item.get('code', '---')}"
                )
            except Exception:
                pass


# ==========================================
# ارسال درخواست فروش برای ادمین
# ==========================================

async def send_sale_to_admin(message, user_id):
    item = create_inventory(message, user_id)

    if item is None:
        await bot.send_message(
            chat_id=get_chat_id(message),
            text=tr(user_id, "no_content")
        )
        return

    sale_id = uuid.uuid4().hex[:12].upper()

    sale = {
        "sale_id": sale_id,
        "user_id": str(user_id),
        "chat_id": get_chat_id(message),
        "username": "",
        "name": "",
        "item": item
    }

    data["pending_sales"].append(sale)
    save_data()

    admin_text = (
        f"{tr(user_id, 'new_sale')}\n\n"
        f"شناسه درخواست: {sale_id}\n"
        f"شناسه کاربر: {user_id}\n"
        f"کد پیشنهادی: {item['code']}\n"
        f"نوع پیام: {item['type']}\n\n"
        f"محتوا:\n{item.get('caption') or 'بدون متن'}"
    )

    buttons = inline_keyboard([
        [
            (f"sale_accept:{sale_id}", tr(user_id, "accept")),
            (f"sale_reject:{sale_id}", tr(user_id, "reject"))
        ],
        [
            (f"sale_wait:{sale_id}", tr(user_id, "wait_button"))
        ]
    ])

    try:
        await bot.send_message(
            chat_id=str(ADMIN_ID),
            text=admin_text,
            inline_keypad=buttons
        )

        if item["type"] != "text":
            await send_media(ADMIN_ID, item)

        await bot.send_message(
            chat_id=get_chat_id(message),
            text=tr(user_id, "sent")
        )

        states.pop(str(user_id), None)

    except Exception as error:
        print("Sale request error:", repr(error))

        await bot.send_message(
            chat_id=get_chat_id(message),
            text=tr(user_id, "error")
        )


# ==========================================
# افزودن و حذف موجودی
# ==========================================

async def admin_add_inventory(message, user_id):
    item = create_inventory(message, user_id)

    if item is None:
        await bot.send_message(
            chat_id=get_chat_id(message),
            text=tr(user_id, "no_content")
        )
        return

    data["inventory"].append(item)
    save_data()

    await bot.send_message(
        chat_id=get_chat_id(message),
        text=(
            f"{tr(user_id, 'added')}\n"
            f"کد موجودی: {item['code']}"
        )
    )

    states.pop(str(user_id), None)


async def admin_remove_inventory(message, user_id):
    search_text = get_message_content(message).strip()

    if not search_text:
        await bot.send_message(
            chat_id=get_chat_id(message),
            text=tr(user_id, "not_found")
        )
        return

    search_lower = search_text.lower()

    for index, item in enumerate(data["inventory"]):
        code = str(item.get("code", "")).lower()
        caption = str(item.get("caption", "")).lower()

        if search_lower == code or search_lower == caption:
            removed = data["inventory"].pop(index)
            save_data()

            states.pop(str(user_id), None)

            await bot.send_message(
                chat_id=get_chat_id(message),
                text=(
                    f"{tr(user_id, 'removed')}\n"
                    f"کد موجودی حذف‌شده: {removed.get('code', '---')}"
                ),
                chat_keypad=admin_keyboard()
            )
            return

    await bot.send_message(
        chat_id=get_chat_id(message),
        text=tr(user_id, "not_found")
    )


# ==========================================
# بررسی دکمه‌های ادمین و انتخاب زبان
# ==========================================

@bot.callback_query_handler()
async def callback_handler(client, message):
    try:
        button_id = str(
            getattr(message, "button_id", None)
            or getattr(message, "data", None)
            or ""
        )

        user_id = get_user_id(message)
        chat_id = get_chat_id(message)

        # انتخاب زبان
        if button_id in ("lang_fa", "lang_en"):
            language = button_id.replace("lang_", "")

            data["users"].setdefault(str(user_id), {})
            data["users"][str(user_id)]["language"] = language
            save_data()

            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "welcome"),
                chat_keypad=user_keyboard(user_id)
            )
            return

        # فقط ادمین اجازه بررسی درخواست‌ها را دارد
        if not is_admin(message):
            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "only_admin")
            )
            return

        # تشخیص عملیات و شناسه درخواست
        parts = button_id.split(":", 1)

        if len(parts) != 2:
            return

        action, sale_id = parts

        sale = next(
            (
                entry
                for entry in data["pending_sales"]
                if entry.get("sale_id") == sale_id
            ),
            None
        )

        if sale is None:
            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "already_done")
            )
            return

        target_user = str(sale["user_id"])

        # تأیید فروش
        if action == "sale_accept":
            item = sale["item"]

            item["code"] = create_code()
            data["inventory"].append(item)

            data["pending_sales"].remove(sale)
            save_data()

            await bot.send_message(
                chat_id=target_user,
                text=(
                    tr(target_user, "accepted")
                    + f"\n\nکد موجودی: {item['code']}"
                )
            )

            await bot.send_message(
                chat_id=chat_id,
                text=f"✅ درخواست {sale_id} تأیید شد."
            )

        # رد فروش
        elif action == "sale_reject":
            data["pending_sales"].remove(sale)
            save_data()

            await bot.send_message(
                chat_id=target_user,
                text=tr(target_user, "rejected")
            )

            await bot.send_message(
                chat_id=chat_id,
                text=f"⭕️ درخواست {sale_id} رد شد."
            )

        # درخواست آیدی از کاربر
        elif action == "sale_wait":
            states[target_user] = {
                "mode": "waiting_user_id",
                "sale_id": sale_id
            }

            await bot.send_message(
                chat_id=target_user,
                text=tr(target_user, "wait")
            )

            await bot.send_message(
                chat_id=chat_id,
                text=f"🆔 درخواست آیدی برای کاربر {target_user} ارسال شد."
            )

    except Exception as error:
        print("Callback error:", repr(error))


# ==========================================
# دریافت پیام‌های کاربران و ادمین
# ==========================================

@bot.on_message()
async def handle_message(client, message):
    try:
        user_id = get_user_id(message)
        chat_id = get_chat_id(message)
        content = get_message_content(message)

        # ثبت اولیه کاربر
        data["users"].setdefault(str(user_id), {})
        save_data()

        # دستور /myid
        if content.split()[:1] == ["/myid"]:
            await bot.send_message(
                chat_id=chat_id,
                text=(
                    f"{tr(user_id, 'myid')}: {user_id}\n"
                    f"{tr(user_id, 'chatid')}: {chat_id}"
                )
            )
            return

        # دستور /start
        if content.split()[:1] == ["/start"]:
            if is_admin(message):
                await bot.send_message(
                    chat_id=chat_id,
                    text=tr(user_id, "admin"),
                    chat_keypad=admin_keyboard()
                )
            else:
                await bot.send_message(
                    chat_id=chat_id,
                    text=tr(user_id, "language"),
                    inline_keypad=inline_keyboard([
                        [
                            ("lang_fa", "🇮🇷 فارسی"),
                            ("lang_en", "🇬🇧 English")
                        ]
                    ])
                )
            return

        # اگر پیام از ادمین باشد
        if is_admin(message):
            if content == "افزودن موجودی ✅":
                states[str(user_id)] = {"mode": "admin_add"}

                await bot.send_message(
                    chat_id=chat_id,
                    text=tr(user_id, "admin_add")
                )
                return

            if content == "لغو کردن یکی از موجودی ها ⭕️":
                states[str(user_id)] = {"mode": "admin_remove"}

                await bot.send_message(
                    chat_id=chat_id,
                    text=tr(user_id, "admin_remove")
                )
                return

        # بررسی وضعیت کاربر
        state = states.get(str(user_id), {})
        mode = state.get("mode")

        # افزودن موجودی توسط ادمین
        if is_admin(message) and mode == "admin_add":
            await admin_add_inventory(message, user_id)
            return

        # حذف موجودی توسط ادمین
        if is_admin(message) and mode == "admin_remove":
            await admin_remove_inventory(message, user_id)
            return

        # دریافت آیدی کاربر
        if mode == "waiting_user_id":
            sale_id = state.get("sale_id")

            await bot.send_message(
                chat_id=str(ADMIN_ID),
                text=(
                    "🆔 آیدی کاربر دریافت شد.\n\n"
                    f"شناسه کاربر: {user_id}\n"
                    f"شناسه درخواست: {sale_id}\n"
                    f"آیدی ارسالی:\n{content}"
                )
            )

            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "id_sent")
            )

            states.pop(str(user_id), None)
            return

        # فروش موجودی توسط کاربر
        if mode == "selling":
            await send_sale_to_admin(message, user_id)
            return

        # فهرست موجودی
        if (
            "لیست موجودی" in content
            or content == "Inventory List"
        ):
            await show_inventory(chat_id, user_id)
            return

        # شروع فروش
        if (
            "فروش موجودی" in content
            or content == "Selling inventory to admin"
        ):
            states[str(user_id)] = {"mode": "selling"}

            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "sell")
            )
            return

        # برگشت
        if content in ("برگشت 🔙", "Back 🔙"):
            states.pop(str(user_id), None)

            await show_home(chat_id, user_id)
            return

        # پیام‌های ناشناخته از ادمین
        if is_admin(message):
            await bot.send_message(
                chat_id=chat_id,
                text=tr(user_id, "choose"),
                chat_keypad=admin_keyboard()
            )

    except Exception as error:
        print("Message handler error:", repr(error))

        try:
            await bot.send_message(
                chat_id=get_chat_id(message),
                text=tr(get_user_id(message), "error")
            )
        except Exception as secondary_error:
            print("Error reply failed:", repr(secondary_error))


# ==========================================
# اجرای ربات
# ==========================================

if __name__ == "__main__":
    print("در حال اتصال به روبیکا...")

    try:
        bot.run()

    except KeyboardInterrupt:
        print("ربات متوقف شد.")

    except Exception as error:
        print("خطای اجرای ربات:", repr(error))
