import tkinter as tk
from tkinter import messagebox, Label, Button
from PIL import Image, ImageTk
import random
import pygame

class CoinFlipGame(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Игра Угадай Монетку")
        self.geometry("1920x1080")
        self.configure(bg='black')

        pygame.init()
        self.coin_sound = pygame.mixer.Sound('777.wav')

        self.load_resources()
        self.current_level = 1
        self.attempts_left = 5

        self.show_welcome_screen()

    def load_resources(self):
        self.bg_image = ImageTk.PhotoImage(Image.open('322.png').resize((1920, 1080), ))
        self.coin_images = [ImageTk.PhotoImage(Image.open(f'coin_{i}.png').resize((100, 100), )) for i in range(1, 3)]

    def show_welcome_screen(self):
        self.clear_screen()
        Label(self, text="Добро пожаловать в игру 'Угадай монетку'!", bg='black', fg='white', font=('Helvetica', 24)).pack(pady=(150, 20))
        Button(self, text="Начать", command=self.start_game, bg='white', fg='black', font=('Helvetica', 18)).pack()

    def start_game(self):
        self.current_level = 1
        self.attempts_left = 5
        self.clear_screen()
        self.create_widgets()

    def create_widgets(self):
        Label(self, image=self.bg_image).place(x=0, y=0, relwidth=1, relheight=1)
        Button(self, image=self.coin_images[0], command=lambda: self.guess('орел'), borderwidth=0).place(x=150, y=250)
        Button(self, image=self.coin_images[1], command=lambda: self.guess('решка'), borderwidth=0).place(x=550, y=250)

    def play_sound(self):
        self.coin_sound.play()

    def guess(self, choice):
        self.play_sound()
        self.animate_coin(choice)

    def animate_coin(self, choice):
        self.coin_label = Label(self, bg='black')
        self.coin_label.pack(expand=True)
        self.update_coin_animation(0, len(self.coin_images) * 5, choice)

    def update_coin_animation(self, current_frame, total_frames, choice):
        if current_frame < total_frames:
            self.coin_label.configure(image=self.coin_images[current_frame % len(self.coin_images)])
            current_frame += 1
            self.after(100, lambda: self.update_coin_animation(current_frame, total_frames, choice))
        else:
            self.coin_label.pack_forget()
            self.show_result(choice)

    def show_result(self, choice):
        result = random.choice(['орел', 'решка'])
        if choice == result:
            self.advance_level()
        else:
            self.attempts_left -= 1
            if self.attempts_left > 0:
                messagebox.showwarning("Неправильно", f"Не угадали. Осталось попыток: {self.attempts_left}.")
            else:
                self.game_over()

    def advance_level(self):
        if self.current_level == 3:
            messagebox.showinfo("Победа", "Поздравляем! Вы прошли эту игру.\nХорошо было провести с вами время. До свидания!")
            self.after(3000, self.quit_game)
        else:
            self.current_level += 1
            self.attempts_left = {2: 3, 3: 1}[self.current_level]
            messagebox.showinfo("Уровень пройден", f"Поздравляем! Вы переходите на уровень {self.current_level}. У вас {self.attempts_left} попытки.")
            self.clear_screen()
            self.create_widgets()

    def game_over(self):
        self.clear_screen()
        Button(self, text="Начать сначала", command=self.restart_game, bg='white', fg='black', font=('Helvetica', 18)).pack(pady=(20, 10))
        Button(self, text="Выйти из игры", command=self.quit_game, bg='white', fg='black', font=('Helvetica', 18)).pack(pady=(0, 20))

    def restart_game(self):
        self.show_welcome_screen()

    def quit_game(self):
        self.destroy()

    def clear_screen(self):
        for widget in self.winfo_children():
            widget.pack_forget()

if __name__ == "__main__":
    app = CoinFlipGame()
    app.mainloop()