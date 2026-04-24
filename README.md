import json
import os
from tkinter import *
from tkinter import ttk, messagebox

DATA_FILE = "books.json"

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker — Трекер прочитанных книг")
        self.root.geometry("800x500")
        self.books = []
        self.load_data()
        # Виджеты ввода
        input_frame = LabelFrame(root, text="Добавить новую книгу", padx=10, pady=10)
        input_frame.pack(pady=10, padx=10, fill="x")
        Label(input_frame, text="Название:").grid(row=0, column=0, sticky="w")
        self.title_entry = Entry(input_frame, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=2)
        Label(input_frame, text="Автор:").grid(row=1, column=0, sticky="w")
        self.author_entry = Entry(input_frame, width=30)
        self.author_entry.grid(row=1, column=1, padx=5, pady=2)
        Label(input_frame, text="Жанр:").grid(row=2, column=0, sticky="w")
        self.genre_entry = Entry(input_frame, width=30)
        self.genre_entry.grid(row=2, column=1, padx=5, pady=2)
        Label(input_frame, text="Кол-во страниц:").grid(row=3, column=0, sticky="w")
        self.pages_entry = Entry(input_frame, width=30)
        self.pages_entry.grid(row=3, column=1, padx=5, pady=2)
        add_btn = Button(input_frame, text="Добавить книгу", command=self.add_book, bg="lightblue")
        add_btn.grid(row=4, column=0, columnspan=2, pady=10)
        # Фильтры
        filter_frame = LabelFrame(root, text="Фильтрация", padx=10, pady=10)
        filter_frame.pack(pady=5, padx=10, fill="x")
        Label(filter_frame, text="Фильтр по жанру:").grid(row=0, column=0, sticky="w")
        self.filter_genre_var = StringVar()
        self.filter_genre_entry = Entry(filter_frame, textvariable=self.filter_genre_var, width=20)
        self.filter_genre_entry.grid(row=0, column=1, padx=5)
        Label(filter_frame, text="Страниц больше:").grid(row=1, column=0, sticky="w")
        self.filter_pages_var = StringVar()
        self.filter_pages_entry = Entry(filter_frame, textvariable=self.filter_pages_var, width=10)
        self.filter_pages_entry.grid(row=1, column=1, sticky="w", padx=5)
        filter_btn = Button(filter_frame, text="Применить фильтр", command=self.apply_filter)
        filter_btn.grid(row=2, column=0, columnspan=2, pady=5)
        reset_btn = Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter)
        reset_btn.grid(row=3, column=0, columnspan=2, pady=5)
        # Таблица с книгами
        columns = ("Название", "Автор", "Жанр", "Страницы")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        self.tree.pack(pady=10, padx=10, fill="both", expand=True)
        # Кнопка сохранения вручную (автосохранение при добавлении)
        save_btn = Button(root, text="Сохранить в JSON", command=self.save_data)
        save_btn.pack(pady=5)
        # Отображаем все книги
        self.display_books(self.books)
    # Валидация полей
    def validate_inputs(self, title, author, genre, pages):
        if not title or not author or not genre:
            messagebox.showerror("Ошибка", "Все поля, кроме страниц, должны быть заполнены!")
            return False
        try:
            pages_int = int(pages)
            if pages_int <= 0:
                raise ValueError
        except ValueError:
            messagebox.showerror("Ошибка", "Количество страниц должно быть положительным числом!")
            return False
        return True
    # Добавление книги
    def add_book(self):
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_entry.get().strip()
        pages = self.pages_entry.get().strip()
        if not self.validate_inputs(title, author, genre, pages):
            return
        new_book = {
            "title": title,
            "author": author,
            "genre": genre,
            "pages": int(pages)
        }
        self.books.append(new_book)
        self.save_data() # автосохранение
        self.display_books(self.books)
        self.clear_entries()
    # Отображение списка книг в Treeview
    def display_books(self, book_list):
        for row in self.tree.get_children():
            self.tree.delete(row)
        for book in book_list:
            self.tree.insert("", END, values=(book["title"], book["author"], book["genre"], book["pages"]))
    # Применить фильтры
    def apply_filter(self):
        genre_filter = self.filter_genre_var.get().strip().lower()
        pages_filter = self.filter_pages_var.get().strip()
        filtered = self.books[:]
        if genre_filter:
            filtered = [b for b in filtered if genre_filter in b["genre"].lower()]
        if pages_filter:
            try:
                pages_limit = int(pages_filter)
                filtered = [b for b in filtered if b["pages"] > pages_limit]
            except ValueError:
                messagebox.showerror("Ошибка", "Фильтр по страницам должен быть числом!")
                return
        self.display_books(filtered)
    # Сброс фильтров
    def reset_filter(self):
        self.filter_genre_var.set("")
        self.filter_pages_var.set("")
        self.display_books(self.books)
    # Очистка полей ввода
    def clear_entries(self):
        self.title_entry.delete(0, END)
        self.author_entry.delete(0, END)
        self.genre_entry.delete(0, END)
        self.pages_entry.delete(0, END)
    # Сохранение в JSON
    def save_data(self):
        try:
            with open(DATA_FILE, "w", encoding="utf-8") as f:
                json.dump(self.books, f, ensure_ascii=False, indent=4)
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить файл:\n{e}")
    # Загрузка из JSON
    def load_data(self):
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, "r", encoding="utf-8") as f:
                    self.books = json.load(f)
            except (json.JSONDecodeError, FileNotFoundError):
                self.books = []
        else:
            self.books = []

if __name__ == "__main__":
    root = Tk()
    app = BookTracker(root)
    root.mainloop()
