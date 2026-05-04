Код GUI-приложения "Random Password Generator"

import tkinter as tk
from tkinter import ttk, messagebox, scrolledtext
import random
import string
import json
from datetime import datetime
import os

# --- Константы ---
MIN_PASSWORD_LENGTH = 4
MAX_PASSWORD_LENGTH = 32
HISTORY_FILE = "password_history.json"

# --- Основной класс приложения ---
class PasswordGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Password Generator")
        self.root.geometry("700x550") # Увеличил размер окна для истории
        self.root.resizable(False, False) # Ограничил изменение размера окна

        self.history = [] # Список для хранения истории паролей
        self.load_history() # Загружаем историю при запуске

        # --- Стили ---
        style = ttk.Style()
        style.configure("TButton", padding=6, relief="flat", font=('Helvetica', 10))
        style.configure("TCheckbutton", font=('Helvetica', 10))
        style.configure("TLabel", font=('Helvetica', 10))
        style.configure("TFrame", padding=10)

        # --- Переменные для чекбоксов ---
        self.use_digits = tk.BooleanVar(value=True)
        self.use_lower = tk.BooleanVar(value=True)
        self.use_upper = tk.BooleanVar(value=True)
        self.use_special = tk.BooleanVar(value=False)

        # --- Фрейм для настроек ---
        settings_frame = ttk.Frame(root)
        settings_frame.pack(pady=10, fill=tk.X)

        # --- Ползунок длины пароля ---
        length_label = ttk.Label(settings_frame, text="Длина пароля:")
        length_label.grid(row=0, column=0, padx=5, pady=5, sticky=tk.W)

        self.length_var = tk.IntVar(value=12) # Значение по умолчанию
        self.length_slider = ttk.Scale(settings_frame, from_=MIN_PASSWORD_LENGTH, to=MAX_PASSWORD_LENGTH, orient=tk.HORIZONTAL,
                                       variable=self.length_var, command=self.update_length_label)
        self.length_slider.grid(row=0, column=1, padx=5, pady=5, sticky=tk.EW)

        self.length_value_label = ttk.Label(settings_frame, text=f"{self.length_var.get()}")
        self.length_value_label.grid(row=0, column=2, padx=5, pady=5, sticky=tk.W)
        settings_frame.grid_columnconfigure(1, weight=1) # Растягиваем ползунок

        # --- Чекбоксы для выбора символов ---
        checkbox_frame = ttk.Frame(settings_frame)
        checkbox_frame.grid(row=1, column=0, columnspan=3, pady=5, sticky=tk.W)

        digits_check = ttk.Checkbutton(checkbox_frame, text="Цифры", variable=self.use_digits)
        digits_check.grid(row=0, column=0, padx=5)

        lower_check = ttk.Checkbutton(checkbox_frame, text="Строчные буквы", variable=self.use_lower)
        lower_check.grid(row=0, column=1, padx=5)

        upper_check = ttk.Checkbutton(checkbox_frame, text="Заглавные буквы", variable=self.use_upper)
        upper_check.grid(row=0, column=2, padx=5)

        special_check = ttk.Checkbutton(checkbox_frame, text="Спецсимволы", variable=self.use_special)
        special_check.grid(row=0, column=3, padx=5)

        # --- Кнопка генерации ---
        generate_button = ttk.Button(settings_frame, text="Сгенерировать пароль", command=self.generate_password)
        generate_button.grid(row=2, column=0, columnspan=3, pady=10)

        # --- Поле для вывода сгенерированного пароля ---
        result_frame = ttk.Frame(root)
        result_frame.pack(pady=5, fill=tk.X)

        result_label = ttk.Label(result_frame, text="Сгенерированный пароль:")
        result_label.pack(side=tk.LEFT, padx=10)

        self.generated_password_entry = ttk.Entry(result_frame, width=40, font=('Helvetica', 12, 'bold'), state='readonly')
        self.generated_password_entry.pack(side=tk.LEFT, padx=10, fill=tk.X, expand=True)

        # --- Фрейм для истории ---
        history_frame = ttk.Labelframe(root, text="История паролей", padding=(10, 5))
        history_frame.pack(pady=10, padx=10, fill=tk.BOTH, expand=True)

        self.history_text = scrolledtext.ScrolledText(history_frame, wrap=tk.WORD, width=80, height=15, font=('Consolas', 10))
        self.history_text.pack(fill=tk.BOTH, expand=True)
        self.history_text.config(state=tk.DISABLED) # Только для чтения

        self.load_history_display() # Загружаем отображение истории

        # --- Обработка закрытия окна ---
        self.root.protocol("WM_DELETE_WINDOW", self.on_closing)

    def update_length_label(self, value):
        """Обновляет метку с текущей длиной пароля."""
        self.length_value_label.config(text=f"{int(float(value))}")

    def generate_password(self):
        """Генерирует пароль на основе выбранных настроек."""
        length = self.length_var.get()

        # Проверка корректности ввода
        if not (MIN_PASSWORD_LENGTH <= length <= MAX_PASSWORD_LENGTH):
            messagebox.showerror("Ошибка", f"Длина пароля должна быть от {MIN_PASSWORD_LENGTH} до {MAX_PASSWORD_LENGTH} символов.")
            return

        char_set = ""
        if self.use_digits.get():
            char_set += string.digits
        if self.use_lower.get():
            char_set += string.ascii_lowercase
        if self.use_upper.get():
            char_set += string.ascii_uppercase
        if self.use_special.get():
            char_set += string.punctuation

        if not char_set: # Если ни один тип символов не выбран
            messagebox.showwarning("Предупреждение", "Выберите хотя бы один тип символов для генерации пароля.")
            return

        # Генерация пароля
        password = ''.join(random.choice(char_set) for _ in range(length))

        # Добавление в историю
        password_info = {
            "password": password,
            "length": length,
            "digits": self.use_digits.get(),
            "lower": self.use_lower.get(),
            "upper": self.use_upper.get(),
            "special": self.use_special.get(),
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        }
        self.history.insert(0, password_info) # Добавляем в начало списка

        # Обновление поля с паролем
        self.generated_password_entry.config(state='normal')
        self.generated_password_entry.delete(0, tk.END)
        self.generated_password_entry.insert(0, password)
        self.generated_password_entry.config(state='readonly')

        self.update_history_display() # Обновляем отображение истории

    def load_history(self):
        """Загружает историю паролей из JSON файла."""
        if os.path.exists(HISTORY_FILE):
            try:
                with open(HISTORY_FILE, 'r', encoding='utf-8') as f:
                    self.history = json.load(f)
                    # Важно: JSON хранит строки, поэтому boolean значения могут быть прочитаны как строки.
                    # Для корректного использования в дальнейшем, можно преобразовать их обратно.
                    for item in self.history:
                        item['digits'] = bool(item.get('digits', False))
                        item['lower'] = bool(item.get('lower', False))
                        item['upper'] = bool(item.get('upper', False))
                        item['special'] = bool(item.get('special', False))
            except (json.JSONDecodeError, IOError) as e:
                messagebox.showerror("Ошибка загрузки истории", f"Не удалось загрузить историю из {HISTORY_FILE}:\n{e}\nСоздана новая история.")
                self.history = [] # Очищаем, если файл поврежден
        else:
            self.history = [] # Файл не существует, начинаем с пустой истории

    def save_history(self):
        """Сохраняет текущую историю паролей в JSON файл."""
        try:
            with open(HISTORY_FILE, 'w', encoding='utf-8') as f:
                json.dump(self.history, f, indent=4, ensure_ascii=False)
        except IOError as e:
            messagebox.showerror("Ошибка сохранения истории", f"Не удалось сохранить историю в {HISTORY_FILE}:\n{e}")

    def update_history_display(self):
        """Обновляет текстовое поле истории."""
        self.history_text.config(state=tk.NORMAL)
        self.history_text.delete(1.0, tk.END) # Очищаем текущее содержимое

        if not self.history:
            self.history_text.insert(tk.END, "История пока пуста.")
        else:
            # Отображаем в обратном порядке (последний вверху)
            for i, entry in enumerate(reversed(self.history)):
                self.history_text.insert(tk.END, f"{i+1}. ")
                self.history_text.insert(tk.END, f"Пароль: {entry['password']}\n", ('bold',)) # Выделяем пароль
                self.history_text.insert(tk.END, f"   Длина: {entry['length']}\n")
                settings = []
                if entry.get('digits'): settings.append("Цифры")
                if entry.get('lower'): settings.append("Строчные")
                if entry.get('upper'): settings.append("Заглавные")
                if entry.get('special'): settings.append("Спецсимволы")
                self.history_text.insert(tk.END, f"   Настройки: {', '.join(settings)}\n")
                self.history_text.insert(tk.END, f"   Время: {entry['timestamp']}\n\n")

        self.history_text.config(state=tk.DISABLED)

        # Применяем тег для выделения пароля
        self.history_text.tag_config('bold', font=('Consolas', 10, 'bold'))

    def load_history_display(self):
        """Загружает историю в виджет ScrolledText при инициализации."""
        self.update_history_display()

    def on_closing(self):
        """Вызывается при закрытии окна, сохраняет историю."""
        self.save_history()
        self.root.destroy()

# --- Запуск приложения ---
if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGeneratorApp(root)




    root.mainloop()

# Random Password Generator (Генератор случайных паролей)

## Автор

** Бирюков Андрей Андреевич **

---

## Описание

Приложение "Random Password Generator" представляет собой графический интерфейс (GUI) на Python, предназначенный для генерации надежных случайных паролей. Пользователь может настраивать различные параметры пароля, включая:

*   **Длину пароля:** С помощью ползунка можно выбрать желаемую длину от 4 до 32 символов.
*   **Типы символов:** Предусмотрены чекбоксы для включения/исключения в пароле:
    *   Цифры
    *   Строчные буквы (a-z)
    *   Заглавные буквы (A-Z)
    *   Специальные символы (!@#$%^&*()_+-=[]{}|;':",./<>?)

Приложение также ведет историю сгенерированных паролей, сохраняя каждый сгенерированный пароль, его настройки и время генерации. История автоматически загружается при запуске приложения и сохраняется в файл `password_history.json` при его закрытии.

---

## Установка и запуск

1.  **Клонирование репозитория (если используете Git):**
    ```bash
    git clone https://github.com/ваш_логин/ваш_репозиторий.git
    cd ваш_репозиторий
    ```
    (Замените URL на актуальный URL вашего репозитория)

2.  **Установка зависимостей (если требуются, в данном случае - стандартная библиотека Python):**
    Данное приложение использует только стандартные библиотеки Python (`tkinter`, `random`, `string`, `json`, `datetime`, `os`), поэтому дополнительные зависимости устанавливать не нужно. Убедитесь, что у вас установлен Python 3.x.

3.  **Запуск приложения:**
    Откройте терминал или командную строку, перейдите в папку с файлами проекта и выполните команду:
    ```bash
    python random_password_generator.py
    ```
    (Предполагается, что основной файл приложения назван `random_password_generator.py`)

---

## Примеры использования

1.  **Генерация стандартного пароля:**
    *   При запуске приложения предлагается длина пароля по умолчанию (например, 12 символов).
    *   Выбраны все типы символов (цифры, строчные, заглавные).
    *   Нажмите кнопку "Сгенерировать пароль".
    *   В поле "Сгенерированный пароль" появится новый пароль, например: `aB3fG7hJk9lP`.
    *   Этот пароль будет также добавлен в "Историю паролей".

2.  **Генерация более сложного пароля:**
    *   Передвиньте ползунок на максимальное значение (32).
    *   Установите чекбоксы "Цифры", "Строчные буквы", "Заглавные буквы" и "Спецсимволы".
    *   Нажмите "Сгенерировать пароль".
    *   Будет сгенерирован длинный и надежный пароль.

3.  **Генерация пароля только из цифр:**
    *   Установите ползунок на желаемую длину (например, 8).
    *   Отметьте только чекбокс "Цифры".
    *   Снимите галочки с остальных чекбоксов.
    *   Нажмите "Сгенерировать пароль".
    *   Вы получите пароль, состоящий только из цифр, например: `84720193`.

4.  **Просмотр истории:**
    *   Сгенерируйте несколько паролей с разными настройками.
    *   Прокрутите раздел "История паролей". Вы увидите список всех ранее сгенерированных паролей, их длину, использованные типы символов и время генерации.

5.  **Проверка минимальной/максимальной длины:**
    *   Попытайтесь вручную изменить значение длины (если бы поле было редактируемым) или ограничить работу ползунка (в текущей реализации ползунок ограничен). Программа не позволит создать пароль длиной менее 4 или более 32 символов, выводя соответствующее сообщение об ошибке.

---

## Возможные тесты (по желанию)

*   **Тест 1: Полная генерация:**
    *   Установить максимальную длину (32).
    *   Включить все типы символов.
    *   Сгенерировать пароль.
    *   Проверить, что длина сгенерированного пароля равна 32.
    *   Проверить, что пароль содержит цифры, строчные, заглавные буквы и спецсимволы.
*   **Тест 2: Генерация только цифр:**
    *   Установить длину 10.
    *   Включить только "Цифры".
    *   Отключить все остальные типы.
    *   Сгенерировать пароль.
    *   Проверить, что пароль имеет длину 10 и состоит только из цифр.
*   **Тест 3: Отсутствие выбора символов:**
    *   Снять галочки со всех чекбоксов.
    *   Нажать "Сгенерировать пароль".
    *   Проверить, что появилось соответствующее предупреждение и пароль не был сгенерирован.
*   **Тест 4: Сохранение и Загрузка истории:**
    *   Сгенерировать несколько паролей.
    *   Закрыть приложение.
    *   Запустить приложение снова.
    *   Проверить, что история предыдущих паролей загрузилась корректно.
    *   Проверить наличие файла `password_history.json` в директории проекта.

---
