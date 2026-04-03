import turtle
import math
import random

# Настройки окна
wn = turtle.Screen()
wn.bgcolor("black")
wn.title("Space Shooter - Управление WASD")
wn.setup(width=800, height=600)
wn.tracer(0)

# Переменные игры
score = 0
level = 1
enemies_per_level = 5
enemies_defeated = 0
game_over = False

# Класс игрока
class Player(turtle.Turtle):
    def __init__(self):
        super().__init__()
        self.shape("triangle")
        self.color("white")
        self.penup()
        self.speed(0)
        self.goto(0, -250)
        self.setheading(90)
        self.speed_x = 0

    def move_left(self):
        self.speed_x = -3

    def move_right(self):
        self.speed_x = 3

    def stop(self):
        self.speed_x = 0

    def move(self):
        self.setx(self.xcor() + self.speed_x)
        if self.xcor() > 350:
            self.setx(350)
        if self.xcor() < -350:
            self.setx(-350)

# Класс снаряда
class Bullet(turtle.Turtle):
    def __init__(self):
        super().__init__()
        self.shape("circle")
        self.color("yellow")
        self.shapesize(0.5, 0.5)
        self.penup()
        self.speed(0)
        self.setheading(90)
        self.goto(0, -240)
        self.state = "ready"

    def fire(self):
        if self.state == "ready":
            self.state = "fire"
            self.goto(player.xcor(), player.ycor() + 10)
            self.showturtle()

    def move(self):
        if self.state == "fire":
            self.sety(self.ycor() + 5)
            if self.ycor() > 280:
                self.hideturtle()
                self.state = "ready"

# Класс врага
class Enemy(turtle.Turtle):
    def __init__(self, x, y):
        super().__init__()
        self.shape("circle")
        self.color("red")
        self.shapesize(1.2, 1.2)
        self.penup()
        self.speed(0)
        self.goto(x, y)
        self.speed_x = random.choice([-2, -1, 1, 2])
        self.speed_y = 0.5

    def move(self):
        self.setx(self.xcor() + self.speed_x)
        self.sety(self.ycor() - self.speed_y)
        if self.xcor() > 340:
            self.speed_x = -abs(self.speed_x)
        if self.xcor() < -340:
            self.speed_x = abs(self.speed_x)

# Класс меню перезапуска
class RestartMenu:
    def __init__(self):
        self.writer = turtle.Turtle()
        self.writer.hideturtle()
        self.writer.penup()
        self.writer.speed(0)
        self.active = False

    def show(self):
        self.active = True
        self.writer.clear()
        self.writer.color("white")
        self.writer.goto(0, 50)
        self.writer.write("GAME OVER", align="center", font=("Arial", 36, "bold"))
        self.writer.goto(0, 0)
        self.writer.write("Нажми R для перезапуска", align="center", font=("Arial", 18, "normal"))
        self.writer.goto(0, -30)
        self.writer.write("Нажми Q для выхода", align="center", font=("Arial", 18, "normal"))

    def hide(self):
        self.active = False
        self.writer.clear()

# Создание объектов
player = Player()
bullet = Bullet()
restart_menu = RestartMenu()

# Функции управления WASD
def move_left():
    player.move_left()

def move_right():
    player.move_right()

def stop_left():
    if player.speed_x < 0:
        player.stop()

def stop_right():
    if player.speed_x > 0:
        player.stop()

def fire():
    bullet.fire()

def restart_game():
    global score, level, enemies_defeated, game_over, enemies
    game_over = False
    score = 0
    level = 1
    enemies_defeated = 0
    restart_menu.hide()

    for enemy in enemies:
        enemy.hideturtle()
        del enemy
    enemies.clear()

    create_enemies(level)

    player.goto(0, -250)
    bullet.hideturtle()
    bullet.state = "ready"
    bullet.goto(0, -240)
    update_score_display()
    update_level_display()

def quit_game():
    wn.bye()

# Функции отображения
def update_score_display():
    score_display.clear()
    score_display.write(f"Счёт: {score}", align="center", font=("Courier", 16, "normal"))

def update_level_display():
    level_display.clear()
    level_display.write(f"Уровень: {level}", align="center", font=("Courier", 16, "normal"))

# Создание врагов
def create_enemies(level):
    num_enemies = enemies_per_level + (level - 1) * 2
    rows = 2 if level < 3 else 3
    cols = (num_enemies + rows - 1) // rows
    start_x = -80 * (cols - 1) / 2
    for row in range(rows):
        for col in range(cols):
            if len(enemies) >= num_enemies:
                break
            x = start_x + col * 80
            y = 250 - row * 60
            enemy = Enemy(x, y)
            enemies.append(enemy)
        if len(enemies) >= num_enemies:
            break

# Проверка коллизий
def check_collisions():
    global score, enemies_defeated, level, game_over
    for enemy in enemies[:]:
        if bullet.distance(enemy) < 20 and bullet.state == "fire":
            bullet.hideturtle()
            bullet.state = "ready"
            enemy.hideturtle()
            enemies.remove(enemy)
            score += 10
            enemies_defeated += 1
            update_score_display()

            if enemies_defeated >= enemies_per_level + (level - 1) * 2:
                level += 1
                enemies_defeated = 0
                update_level_display()
                create_enemies(level)

        if player.distance(enemy) < 25:
            game_over = True
            restart_menu.show()
            return

    if player.ycor() - 20 < -300:
        game_over = True
        restart_menu.show()

# Настройка управления WASD
wn.listen()
wn.onkeypress(move_left, "a")
wn.onkeypress(move_right, "d")
wn.onkeyrelease(stop_left, "a")
wn.onkeyrelease(stop_right, "d")
wn.onkeypress(fire, "w")
wn.onkeypress(restart_game, "r")
wn.onkeypress(quit_game, "q")

# Текст для отображения очков и уровня
score_display = turtle.Turtle()
score_display.color("white")
score_display.penup()
score_display.hideturtle()
score_display.goto(-300, 260)

level_display = turtle.Turtle()
level_display.color("white")
level_display.penup()
level_display.hideturtle()
level_display.goto(300, 260)

enemies = []
create_enemies(level)

update_score_display()
update_level_display()

# Главный игровой цикл
while True:
    wn.update()
    if not game_over:
        player.move()
        bullet.move()

        for enemy in enemies:
            enemy.move()

        check_collisions()
