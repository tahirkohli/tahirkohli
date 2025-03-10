import random
import sqlite3
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters

TOKEN = "7772877855:AAFLV6ciw2VZVQc6jcf6V2GZ6W0kqNx_MQw"

# Database Setup
conn = sqlite3.connect("users.db", check_same_thread=False)
cursor = conn.cursor()
cursor.execute("CREATE TABLE IF NOT EXISTS users (user_id INTEGER PRIMARY KEY, balance INTEGER DEFAULT 100)")
conn.commit()

# Start Command
async def start(update: Update, context):
    user_id = update.message.chat_id
    cursor.execute("INSERT OR IGNORE INTO users (user_id) VALUES (?)", (user_id,))
    conn.commit()
    await update.message.reply_text("🎰 Welcome to Spin Game!\nUse /spin to play (Entry: ₹100)")

# Spin Command
async def spin(update: Update, context):
    user_id = update.message.chat_id
    cursor.execute("SELECT balance FROM users WHERE user_id=?", (user_id,))
    user_data = cursor.fetchone()
    
    if user_data and user_data[0] >= 100:
        cursor.execute("UPDATE users SET balance = balance - 100 WHERE user_id=?", (user_id,))
        conn.commit()

        prize = random.choice([0, 50, 100, 200, 500])  # Random winnings
        cursor.execute("UPDATE users SET balance = balance + ? WHERE user_id=?", (prize, user_id))
        conn.commit()

        await update.message.reply_text(f"🎡 Spin Complete! Aapne jeeta: ₹{prize}\n💰 Aapka naya balance: ₹{user_data[0] + prize - 100}")
    else:
        await update.message.reply_text("❌ Aapke paas ₹100 nahi hain. Pehle /deposit karein!")

# Check Balance
async def balance(update: Update, context):
    user_id = update.message.chat_id
    cursor.execute("SELECT balance FROM users WHERE user_id=?", (user_id,))
    user_data = cursor.fetchone()
    await update.message.reply_text(f"💰 Aapka balance: ₹{user_data[0]}")

app = ApplicationBuilder().token(TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(CommandHandler("spin", spin))
app.add_handler(CommandHandler("balance", balance))

print("Bot chal raha hai...")
app.run_polling()
The official channel for the python-telegram-bot library | https://python-telegram-bot.orgasync def deposit(update: Update, context):
    user_id = update.message.chat_id
    upi_id = "yourupi@upi"  # Apni UPI ID yahan likhein
    await update.message.reply_text(f"📌 Deposit ₹100 or more to this UPI ID:\n\n`{upi_id}`\n\n🔄 Payment karne ke baad screenshot bhejein aur admin approve karega.")
from telegram import InputFile

async def deposit(update: Update, context):
    upi_id = "yourupi@upi"  
    await update.message.reply_photo(photo=InputFile("upi_qr.png"), caption=f"📌 Scan QR ya is UPI ID pe ₹100 send karein:\n\n`{upi_id}`\n\n🔄 Payment hone ke baad admin approve karega.")
import qrcode

upi_id = "officaltahir0@okhdfcbank"  # Apni UPI ID daalein
upi_url = f"upi://pay?pa={upi_id}&pn=YourName&mc=&tid=&tr=&tn=Deposit+for+SpinBot&am=100&cu=INR"

qr = qrcode.make(upi_url)
qr.save("import random
₹100 or more to this UPI ID:\n\n`{upi_id}`\n\n🔄 Payment karne k")

print("✅ UPI QR Code saved as upi_qr.png")
async def withdraw(update: Update, context):
    user_id = update.message.chat_id
    cursor.execute("SELECT balance FROM users WHERE user_id=?", (user_id,))
    user_data = cursor.fetchone()

    if user_data and user_data[0] >= 200:  # Minimum ₹200 withdrawal
        await update.message.reply_text("🔗 Apna UPI ID bhejein (e.g., `yourupi@upi`)")
        context.user_data["awaiting_upi"] = True
    else:
        await update.message.reply_text("❌ Minimum withdrawal ₹200 hai. Pehle balance badhayein!")

async def receive_upi(update: Update, context):
    if context.user_data.get("awaiting_upi"):
        user_id = update.message.chat_id
        upi_id = update.message.text
        cursor.execute("SELECT balance FROM users WHERE user_id=?", (user_id,))
        user_data = cursor.fetchone()

        cursor.execute("UPDATE users SET balance = balance - 200 WHERE user_id=?", (user_id,))
        conn.commit()

        await update.message.reply_text(f"✅ Withdrawal request received!\n🛠 Admin aapko ₹200 bhejega on {upi_id}")
        del context.user_data["awaiting_upi"]
    else:
        await update.message.reply_text("❌ Pehle /withdraw likhein fir UPI ID bhejein.")

app.add_handler(CommandHandler("deposit", deposit))
app.add_handler(CommandHandler("withdraw", withdraw))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, receive_upi))
ADMIN_ID = 123456789  # Apna Telegram User ID daalein

async def add_balance(update: Update, context):
    if update.message.chat_id == ADMIN_ID:
        try:
            user_id = int(context.args[0])
            amount = int(context.args[1])
            cursor.execute("UPDATE users SET balance = balance + ? WHERE user_id=?", (amount, user_id))
            conn.commit()
            await update.message.reply_text(f"✅ {user_id} ka balance ₹{amount} badh gaya!")
        except:
            await update.message.reply_text("❌ Format sahi nahi hai. Use: /addbalance user_id amount")
    else:
        await update.message.reply_text("❌ Aap admin nahi hain!")

async def deduct_balance(update: Update, context):
    if update.message.chat_id == ADMIN_ID:
        try:
            user_id = int(context.args[0])
            amount = int(context.args[1])
            cursor.execute("UPDATE users SET balance = balance - ? WHERE user_id=?", (amount, user_id))
            conn.commit()
            await update.message.reply_text(f"✅ {user_id} ka balance ₹{amount} kam ho gaya!")
        except:
            await update.message.reply_text("❌ Format sahi nahi hai. Use: /deductbalance user_id amount")
    else:
        await update.message.reply_text("❌ Aap admin nahi hain!")

app.add_handler(CommandHandler("addbalance", add_balance))
app.add_handler(CommandHandler("deductbalance", deduct_balance))
