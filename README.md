# zeus-calcio-bot
Bot Telegram analisi calcio
import os
import telebot
import random

TOKEN = os.getenv("BOT_TOKEN")
bot = telebot.TeleBot(TOKEN)

# --- simulazione statistiche realistiche ---
def get_stats(team):
    return {
        "forma": random.uniform(0.3, 0.9),
        "gol_fatti": random.uniform(0.8, 2.5),
        "gol_subiti": random.uniform(0.8, 2.5)
    }

def predict_match(home, away):
    h = get_stats(home)
    a = get_stats(away)

    home_score = (h["forma"] * 50) + (h["gol_fatti"] * 20) - (a["gol_subiti"] * 15)
    away_score = (a["forma"] * 50) + (a["gol_fatti"] * 20) - (h["gol_subiti"] * 15)

    draw_chance = 100 - abs(home_score - away_score)

    total = home_score + away_score + draw_chance

    probs = {
        home: round(home_score / total * 100, 1),
        "X": round(draw_chance / total * 100, 1),
        away: round(away_score / total * 100, 1)
    }

    pick = max(probs, key=probs.get)

    return probs, pick


@bot.message_handler(commands=['start'])
def start(message):
    bot.send_message(message.chat.id, "Bot attivo ⚽\nScrivi: Juventus Milan")


@bot.message_handler(func=lambda m: True)
def handle(message):
    try:
        text = message.text.split()
        if len(text) != 2:
            bot.send_message(message.chat.id, "Scrivi tipo: Juventus Milan")
            return

        home, away = text[0], text[1]

        probs, pick = predict_match(home, away)

        msg = f"""
⚽ {home} vs {away}

📊 Probabilità:
{home}: {probs[home]}%
X: {probs['X']}%
{away}: {probs[away]}%

🔥 Pronostico: {pick}
        """

        bot.send_message(message.chat.id, msg)

    except Exception as e:
        bot.send_message(message.chat.id, "Errore nel calcolo")


bot.polling()