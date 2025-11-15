# alipay-bot
Бот для экономии на Alipay + заработок
from aiogram import Bot, Dispatcher, types, executor
from aiogram.contrib.fsm_storage.memory import MemoryStorage
from aiogram.dispatcher import FSMContext
from aiogram.dispatcher.filters.state import State, StatesGroup
import os

API_TOKEN = os.getenv("BOT_TOKEN")
PAYMENT_URL = "https://google.com"  # ← Заменишь на Paybox

bot = Bot(token=API_TOKEN)
storage = MemoryStorage()
dp = Dispatcher(bot, storage=storage)

class Calculator(StatesGroup):
    waiting_for_sum = State()

def start_keyboard():
    kb = types.InlineKeyboardMarkup()
    kb.add(
        types.InlineKeyboardButton("Рассчитать выгоду", callback_data="calc"),
        types.InlineKeyboardButton("Что входит в курс", callback_data="info")
    )
    return kb

def result_keyboard():
    kb = types.InlineKeyboardMarkup()
    kb.add(
        types.InlineKeyboardButton("Купить курс за 4 990₸", url=PAYMENT_URL),
        types.InlineKeyboardButton("Узнать про заработок", callback_data="earnings")
    )
    return kb

def info_keyboard():
    kb = types.InlineKeyboardMarkup()
    kb.add(types.InlineKeyboardButton("Купить курс за 4 990₸", url=PAYMENT_URL))
    return kb

def earnings_keyboard():
    kb = types.InlineKeyboardMarkup()
    kb.add(types.InlineKeyboardButton("Купить курс за 4 990₸", url=PAYMENT_URL))
    return kb

@dp.message_handler(commands=['start'])
async def cmd_start(message: types.Message):
    text = """
Привет!

Я помогу тебе экономить от 5% на каждом пополнении Alipay!

Курс на площадке: 74.5₸ за 1¥
У посредников: 78.6₸ за 1¥

Экономия: от 4₸ с каждого юаня!

Уже более 30 человек используют наш метод

Бонус: После прохождения курса ты сможешь стать посредником и зарабатывать на переводах для других!

Хочешь узнать, сколько сэкономишь?
"""
    await message.answer(text, reply_markup=start_keyboard())

@dp.callback_query_handler(lambda c: c.data in ['calc', 'info', 'earnings'])
async def process_callback(call: types.CallbackQuery, state: FSMContext):
    if call.data == 'calc':
        await call.message.edit_text(
            "Введи сумму, которую обычно пополняешь (в тенге):\n\nНапример: 100000",
            reply_markup=None
        )
        await Calculator.waiting_for_sum.set()

    elif call.data == 'info':
        text = """
В КУРС ВХОДИТ:

Пошаговая схема пополнения Alipay
Видео-инструкции по каждому шагу
Список 2-3 проверенных продавцов с рейтингом 6000+ сделок
Чек-лист безопасности (12 правил)
Доступ в закрытый чат поддержки навсегда
Обновления курсов бесплатно

БОНУС: Гайд "Как стать посредником и зарабатывать 50-100 тыс₸/мес"

Стоимость: 4 990₸
Акция до конца недели!

После изучения курса ты сможешь не только экономить, но и помогать другим за комиссию 3-5% — это пассивный доход!
"""
        await call.message.edit_text(text, reply_markup=info_keyboard())

    elif call.data == 'earnings':
        text = """
КАК ЗАРАБАТЫВАТЬ КАК ПОСРЕДНИК:

После прохождения курса ты узнаешь всю схему и сможешь:

Помогать другим пополнять Alipay за комиссию 3-5%
Зарабатывать от 50 000₸ до 100 000₸ в месяц
Работать удалённо в удобное время

Пример расчёта:
Если 10 человек в месяц переводят по 300 000₸, твой доход при комиссии 4%:
10 × 300 000₸ × 4% = 120 000₸

В курсе есть отдельный модуль: "Как стать посредником и привлекать клиентов"

Всё что нужно — пройти курс, освоить схему, и начать помогать другим!
"""
        await call.message.edit_text(text, reply_markup=earnings_keyboard())

@dp.message_handler(state=Calculator.waiting_for_sum)
async def process_sum(message: types.Message, state: FSMContext):
    if not message.text.replace(' ', '').isdigit():
        await message.answer("Пожалуйста, введи число (например: 100000)")
        return

    suma = int(message.text.replace(' ', ''))
    await state.update_data(suma=suma)

    platform_yuan = suma / 74.5
    broker_yuan = suma / 78.6
    savings = (platform_yuan - broker_yuan) * 74.5

    text = f"""
ТВОЯ ВЫГОДА:

При сумме {suma:,}₸:

Наша площадка: ~{platform_yuan:,.0f}¥
Посредники: ~{broker_yuan:,.0f}¥

Экономия: ~{savings:,.0f}₸ за раз!

Курс окупится после 2-3 использований!

А ещё: Освоив схему, ты сможешь зарабатывать как посредник для других! Многие наши ученики уже делают 50-100 тыс₸/мес на этом.
"""
    await message.answer(text, reply_markup=result_keyboard())
    await state.finish()

if __name__ == '__main__':
    print("Бот запущен...")
    executor.start_polling(dp, skip_updates=True)
