# Syria
اسعار الدولار
token - 7869834958:AAHT1k10oRd6LCW2K74JOeSTctD-LpITY38
import logging
import os
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext

# إعداد السجل logging لتتبع الأحداث
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    level=logging.INFO)

# قاموس لتخزين نقاط الإحالة لكل مستخدم (لأغراض العرض؛ يفضل استخدام قاعدة بيانات في الإنتاج)
referral_points = {}

# دالة لإرجاع أسعار الصرف (يُستبدل هذا القسم باستدعاء API للحصول على أسعار فعلية)
def get_exchange_rates():
    # قيم افتراضية للتوضيح (قم بتحديثها حسب الحاجة)
    usd_to_syp = "2,500 ل.س"       # مثال: 1 دولار = 2500 ل.س
    eur_to_syp = "2,800 ل.س"       # مثال: 1 يورو = 2800 ل.س
    gold_to_syp = "50,000 ل.س للغرام"  # مثال: سعر الذهب لكل غرام
    message = "💱 أسعار صرف العملات:\n\n"
    message += f"الدولار الأمريكي: {usd_to_syp}\n"
    message += f"اليورو: {eur_to_syp}\n"
    message += f"سعر الذهب: {gold_to_syp}\n"
    return message

# معالج أمر /start
def start(update: Update, context: CallbackContext):
    user_id = update.effective_user.id
    args = context.args

    # فحص ما إذا كان المستخدم دخل مع رمز إحالة (على شكل /start <referrer_id>)
    if args:
        try:
            referrer_id = int(args[0])
            if referrer_id != user_id:  # منع إحالة الذات
                referral_points[referrer_id] = referral_points.get(referrer_id, 0) + 1
                update.message.reply_text("شكراً لاستخدام رابط الإحالة! تم إضافة نقطة للمُحيل.")
            else:
                update.message.reply_text("لا يمكنك استخدام رابطك الخاص للإحالة.")
        except ValueError:
            update.message.reply_text("معرف المُحيل غير صالح.")

    welcome_message = (
        "مرحباً بك في بوت أسعار صرف العملات والذهب!\n\n"
        "الأوامر المتاحة:\n"
        "/rates - عرض أسعار العملات والذهب\n"
        "/join - الانضمام إلى قناة البورصة\n"
        "/referral - عرض رابط الإحالة ونقاطك\n"
        "/withdraw - طلب سحب النقاط\n\n"
        "يمكنك مشاركة رابط الإحالة الخاص بك لكسب نقاط إضافية."
    )
    update.message.reply_text(welcome_message)

# معالج أمر /rates لعرض الأسعار
def rates(update: Update, context: CallbackContext):
    rates_message = get_exchange_rates()
    update.message.reply_text(rates_message)

# معالج أمر /join لعرض رابط قناة البورصة
def join_channel(update: Update, context: CallbackContext):
    join_message = (
        "للانضمام إلى قناة البورصة، اضغط على الرابط التالي:\n"
        "https://t.me/ox_leader1"
    )
    update.message.reply_text(join_message)

# معالج أمر /referral لعرض رابط الإحالة ونقاط المستخدم
def referral(update: Update, context: CallbackContext):
    user_id = update.effective_user.id
    points = referral_points.get(user_id, 0)
    # استبدل YourBotUsername باسم البوت الفعلي الخاص بك
    referral_link = f"https://t.me/YourBotUsername?start={user_id}"
    message = (
        f"🔗 رابط الإحالة الخاص بك:\n{referral_link}\n\n"
        f"✅ نقاطك الحالية: {points} نقطة\n"
        "كل إحالة تمنحك نقطة واحدة.\n"
        "أقل مبلغ للسحب هو 100 نقطة (100 نقطة = 1$)."
    )
    update.message.reply_text(message)

# معالج أمر /withdraw لطلب السحب
def withdraw(update: Update, context: CallbackContext):
    user_id = update.effective_user.id
    points = referral_points.get(user_id, 0)
    if points >= 100:
        dollars = points / 100
        message = (
            f"يمكنك سحب {points} نقطة بما يعادل {dollars}$.\n"
            "يرجى التواصل مع الدعم لإتمام عملية السحب."
        )
    else:
        message = (
            f"عذراً، لا تمتلك نقاطاً كافية للسحب.\n"
            f"نقاطك الحالية: {points} نقطة.\n"
            "الحد الأدنى للسحب هو 100 نقطة (100 نقطة = 1$)."
        )
    update.message.reply_text(message)

def main():
    # الحصول على توكن البوت من متغير البيئة BOT_TOKEN أو استبداله بالتوكن الخاص بك مباشرة
    TOKEN = os.environ.get("BOT_TOKEN", "YOUR_BOT_TOKEN_HERE")
    updater = Updater(TOKEN, use_context=True)
    dp = updater.dispatcher
