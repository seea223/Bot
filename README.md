import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, ContextTypes

BOT_TOKEN = "7533586967:AAGeQHGaJ-OFt952wJM6t7r3YCbTMeWJpVQ"
REF_BONUS = 10

users = {}

logging.basicConfig(level=logging.INFO)

emoji_attack = "⚔"
emoji_defend = "🛡"
emoji_hp = "❤"
emoji_coin = "💰"

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    user_id = user.id
    args = context.args

    if user_id not in users:
        users[user_id] = {"hp": 100, "score": 0, "ref": 0}
        if args:
            ref_id = int(args[0])
            if ref_id != user_id and ref_id in users:
                users[ref_id]["score"] += REF_BONUS
                users[ref_id]["ref"] += 1
                await context.bot.send_message(chat_id=ref_id, text=f"{emoji_coin} Sizga {REF_BONUS} referral ball tushdi!")

    btn = InlineKeyboardMarkup([
        [InlineKeyboardButton(f"{emoji_attack} Jang qilish", callback_data="fight")],
        [InlineKeyboardButton(f"{emoji_coin} Balansni ko'rish", callback_data="balance")],
    ])
    await update.message.reply_text(f"Salom {user.first_name}! Oddiy Askar Jangi o'yiniga hush kelibsiz!", reply_markup=btn)

async def handle_callback(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    user_id = query.from_user.id
    data = query.data
    await query.answer()

    if user_id not in users:
        users[user_id] = {"hp": 100, "score": 0, "ref": 0}

    if data == "fight":
        from random import randint
        dmg = randint(5, 30)
        users[user_id]["hp"] -= dmg
        users[user_id]["score"] += dmg
        msg = f"{emoji_attack} Jangda siz {dmg} zarar oldingiz!\n{emoji_hp} HP: {users[user_id]['hp']}\n{emoji_coin} Ball: {users[user_id]['score']}"
        if users[user_id]["hp"] <= 0:
            users[user_id]["hp"] = 100
            msg += f"\n\n{emoji_defend} Siz qayta tiklandingiz!"
        await query.edit_message_text(msg)

    elif data == "balance":
        u = users[user_id]
        await query.edit_message_text(f"{emoji_coin} Sizning ballaringiz: {u['score']}\n{emoji_hp} HP: {u['hp']}\nReferral: {u['ref']} ta")

if __name__ == '__main__':
    app = ApplicationBuilder().token(BOT_TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CallbackQueryHandler(handle_callback))
    print("Oddiy Askar Jangi bot ishga tushdi!")
    app.run_polling()
