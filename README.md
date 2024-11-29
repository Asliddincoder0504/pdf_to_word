# pdf_to_word
PDF FAYLNI WORD  FAYL QILIB  BERADIGAN  BOT
pdf faylni biz bilan  docx  qiling

import os
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext, CallbackQueryHandler
from pdf2docx import Converter
from dotenv import load_dotenv

# .env fayldan tokenni yuklash
load_dotenv()
BOT_TOKEN = os.getenv("6494372838:AAG-1atdMYSE7KmgWDCsRFuuW_qA9T5jXhU")  # Tokenni o'zingizning .env faylingizdan olish

# /start komandasi
def start(update: Update, context: CallbackContext):
    keyboard = [
        [InlineKeyboardButton("PDF to Word", callback_data='pdf_to_word')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)

    update.message.reply_text(
        "Salom! Men PDF faylini Word formatiga aylantiruvchi botman. Iltimos, PDF faylini yuboring.",
        reply_markup=reply_markup
    )

# PDF faylini Word formatiga aylantirish funksiyasi
def convert_pdf_to_word(pdf_path):
    docx_path = pdf_path.replace(".pdf", ".docx")
    cv = Converter(pdf_path)
    cv.convert(docx_path, start=0, end=None)
    cv.close()
    return docx_path

# Tugmalarni ishlash
def button(update: Update, context: CallbackContext):
    query = update.callback_query
    query.answer()

    if query.data == 'pdf_to_word':
        query.edit_message_text(
            text="Iltimos, PDF faylini yuboring.",
            reply_markup=None
        )

# PDF faylini qabul qilish va konvertatsiya qilish (PDF to Word)
def handle_pdf(update: Update, context: CallbackContext):
    file = update.message.document
    file_id = file.file_id
    file_name = file.file_name

    # Faylni yuklab olish
    new_file = context.bot.get_file(file_id)
    file_path = f"downloads/{file_name}"

    # Agar "downloads" papkasi mavjud bo'lmasa, uni yaratish
    if not os.path.exists("downloads"):
        os.makedirs("downloads")

    # Faylni saqlash
    new_file.download(file_path)

    if file_name.lower().endswith(".pdf"):
        update.message.reply_text("PDF faylini Word formatiga o‘girishni boshlayapman...")

        try:
            # PDF ni DOCX ga aylantirish
            docx_file_path = convert_pdf_to_word(file_path)

            # DOCX faylini foydalanuvchiga yuborish
            update.message.reply_text("Aylantirish tugadi. Word faylini quyida topasiz:")
            context.bot.send_document(chat_id=update.message.chat_id, document=open(docx_file_path, 'rb'))

            # Fayllarni o‘chirib tashlash
            os.remove(file_path)
            os.remove(docx_file_path)
        except Exception as e:
            update.message.reply_text(f"Xatolik yuz berdi: {e}")
    else:
        update.message.reply_text("Iltimos, faqat PDF faylini yuboring.")

# Asosiy bot funksiyasi
def main():
    updater = Updater("6494372838:AAG-1atdMYSE7KmgWDCsRFuuW_qA9T5jXhU")
    dispatcher = updater.dispatcher

    # Handlers
    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(MessageHandler(Filters.document.mime_type("application/pdf"), handle_pdf))
    dispatcher.add_handler(CallbackQueryHandler(button))

    # Botni ishga tushirish
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()

print("Bot ishladi")

