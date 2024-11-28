# Обработчик текста
## Описание
Этот проект представляет собой Jupyter Notebook, предназначенный для обработки текста, особенно полезного для анализа телефонных разговоров. Он включает в себя функции для предварительной обработки текста, такие как удаление стоп-слов, приведение текста к нижнему регистру, объединение реплик одного говорящего и другие преобразования.

## Основные функции
### 1. Импорт библиотек
   ```sh
import pandas as pd
import re
```
### 2. Настройка отображения данных в Pandas
 ```sh
pd.set_option('display.max_columns', None)
pd.set_option('display.max_colwidth', None)
```
### 3. Загрузка данных
 ```sh
df = pd.read_excel("data_call_extended.xlsx")
```
### 4. Функция предварительной обработки текста
 ```sh
def preprocess_conversation(text):
    # Приводим текст к нижнему регистру
    text = text.lower()

    # Заменяем стоп-слова
    stopwords = {
        "ну", "как бы", "типа", "это самое", "значит", "короче", "слушай", "понимаешь",
        "вообще", "на самом деле", "в общем", "просто", "кстати", "эээ", "эм", "ммм",
        "вот", "знаешь", "получается", "собственно", "в принципе", "короче говоря", "так сказать",
        "ясное дело", "ну как бы", "вообще-то", "да, ну", "наверное", "кажется", "честно говоря",
        "в любом случае", "так вот", "между прочим", "кстати говоря", "ну ладно", "точнее",
        "вот это", "прямо", "то есть"
    }
    # Удаляем стоп-слова и лишние символы, кроме меток 1) и 2)
    text = re.sub(r'\\b(?:{})\\b'.format('|'.join(stopwords)), '', text)
    text = re.sub(r'[^\\w\\s\\d():]', '', text)  # Убираем лишние символы

    # Заменяем "1)" на "М:" и "2)" на "К:"
    text = text.replace("1)", "М:").replace("2)", "К:")

    # Объединяем реплики одного говорящего
    result = []
    lines = text.splitlines()  # Разбиваем текст по строкам
    current_speaker = None
    buffer = []

    for line in lines:
        # Разделяем текст на фрагменты по меткам говорящих
        parts = re.split(r'(М:|К:)', line)
        for part in parts:
            # Если найден новый говорящий
            if part in {'М:', 'К:'}:
                # Если сменился говорящий, добавляем буфер в результат
                if current_speaker and buffer and current_speaker != part:
                    result.append(f"{current_speaker} {' '.join(buffer).strip()}")
                    buffer = []
                current_speaker = part
            else:
                # Убираем лишние пробелы и добавляем текст в буфер
                cleaned_part = re.sub(r'\\s+', ' ', part).strip()
                if cleaned_part:
                    buffer.append(cleaned_part)

    # Добавляем последний буфер в результат
    if current_speaker and buffer:
        result.append(f"{current_speaker} {' '.join(buffer).strip()}")

    # Собираем итоговый текст
    processed_text = ' '.join(result)

    return processed_text
```

### 5. Пример использования функции
 ```sh
text = """
1)  Алло, что-то сбросилось. По поводу... Ну как там, это же законодательно у вас должно быть.2)  Алло.2)  Да.2)  Нам не надо.2)  Девушка.2)  Вот, ну,2)  слово нет,2)  оно меняет только одно обозначение. Нет, это нет.2)  Ну, все.1)  Ну, я понимаю, что вы, наверное, не сталкивались пока что.2)  Другого обозначения у этого слова нет.2)  Нам не надо.2)  Да, девушка.2)  У меня опыт,2)  там, где вы учились, я там преподавал.2)  И мне рассказывать,2)  сталкивался, не сталкивался, не надо.
"""

processed_text = preprocess_conversation(text)
print(processed_text)
```

### 6. Цикл обработки
 ```sh
import time

# Создаем пустой список для хранения очищенных текстов
cleaned_texts = []

# Устанавливаем размер чанка
chunk_size = 30

# Переменные для отслеживания времени
total_time = 0  # Общее время обработки
total_chunks = 0  # Общее количество обработанных чанков
total_lines = 0  # Общее количество обработанных строк

# Время обработки каждого чанка
chunk_times = []
line_times = []

# Проходим по DataFrame по 30 строк за раз
for i in range(0, len(df), chunk_size):
    start_time = time.time()  # Засекаем время начала обработки чанка

    chunk = df['text'].iloc[i:i + chunk_size]  # Извлекаем текущий чанк (по 30 строк)

    # Применяем обработчик к каждой строке в текущем чанке и отслеживаем время
    cleaned_chunk = chunk.apply(preprocess_conversation)

    # Добавляем очищенные тексты в список
    cleaned_texts.extend(cleaned_chunk.tolist())

    # Время обработки чанка
    chunk_end_time = time.time()
    chunk_time = chunk_end_time - start_time
    chunk_times.append(chunk_time)

    # Время обработки каждой строки
    line_times.extend([chunk_time / len(chunk)] * len(chunk))

    total_time += chunk_time
    total_chunks += 1
    total_lines += len(chunk)

# Добавляем очищенные тексты в DataFrame как новый столбец 'text_clean'
df['text_clean'] = cleaned_texts

# Вычисляем среднее время обработки чанка и строки
avg_chunk_time = total_time / total_chunks if total_chunks > 0 else 0
avg_line_time = total_time / total_lines if total_lines > 0 else 0

# Выводим статистику
print(f"Общее время обработки: {total_time:.2f} секунд")
print(f"Общее количество обработанных чанков: {total_chunks}")
print(f"Общее количество обработанных строк: {total_lines}")
print(f"Среднее время обработки одного чанка: {avg_chunk_time:.5f} секунд")
print(f"Среднее время обработки одной строки: {avg_line_time:.5f} секунд")
```

## Установка и запуск
### Клонируйте репозиторий:
```sh
git clone https://github.com/ваш-пользователь/ваш-репозиторий.git
```

### Установите необходимые зависимости:
```sh
pip install pandas
```
### Запустите Jupyter Notebook:
```sh
jupyter notebook
```
Откройте файл Обработчик_текста.ipynb и запустите ячейки для обработки текста.

## Лицензия
Этот проект лицензирован под MIT License. Подробности смотрите в файле LICENSE.
