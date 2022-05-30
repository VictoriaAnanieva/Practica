import pygame
from random import randrange as rnd
import time
import threading

pygame.init() #Включение библиотеки

WIDTH, HEIGHT = 1200, 800 #Высота и длинна экрана
fps = 60 #Ограничение фпс
paddle_w, paddle_h, paddle_speed  = 330, 35, 15  #Длинна,Высота и Скорость игрока
ball_radius, ball_speed  = 20, 6 #Радиус, Скорость мяча
ball_rect = int(ball_radius * 2 ** 0.5) #Создание квадрата для формулы вписаного круга
dx, dy = 1, -1 #Множетили для отражение мяча от границ
PlayerX = WIDTH // 2 - paddle_w // 2 #Координаты игрока X
PlayerY = HEIGHT - paddle_h - 10 #Координаты игрока Y
paddle = pygame.Rect(WIDTH // 2 - paddle_w // 2, HEIGHT - paddle_h - 10, paddle_w, paddle_h) #Хранение координат игрока
ball = pygame.Rect(rnd(ball_rect, WIDTH - ball_rect), HEIGHT // 2, ball_rect, ball_rect) #Хранение координат шара
block_list = [pygame.Rect(10 + 120 * i, 10 + 70 * j, 100, 50) for i in range(10) for j in range(4)] #Создание списка с хранением координат прямоугольников
color_list = [(rnd(0,40), rnd(0,40), rnd(30,256)) for i in range(10) for j in range(4)] #Рандомная генерация цвета прямоугольников
data_lock= threading.Lock() #Закрытие данных

Timer = "0" #Таймер для отображение
Timer_Show = 0 #Таймер для подсчета
score, scoreSum  = 0, 0 #Счетчики
end = 1 #Флаг
flag_sound = True #Флаг для музыка
flag = 0 #Флаг для коректного подсчета

sc = pygame.display.set_mode((WIDTH, HEIGHT)) #создане окна
clock = pygame.time.Clock()#Ренулятор кадров

background = pygame.transform.scale (pygame.image.load('textures/background.jpg'), (1200,800)) #Импорт задника
Player = pygame.transform.scale(pygame.image.load('textures/Player.png'), (330, 35)) #Импорт текстуры игрока
pygame.display.set_caption("Защита леса") #Название игры
pygame.display.set_icon(pygame.image.load('textures/logo.bmp')) #Иконка окна
ref = pygame.mixer.Sound('textures/reflection.mp3') #Импорт звука
rem = pygame.mixer.Sound('textures/remove.mp3') #Импорт звука
lose = pygame.mixer.Sound('textures/lose.mp3') #Импорт звука
win_sound = pygame.mixer.Sound('textures/win.mp3') #Импорт звука
pygame.mixer.music.load('textures/music.mp3') #Импорт музыки
pygame.mixer.music.set_volume(0.2) #Контроль громкости звука
lose.set_volume(0.3) #Контроль громкости звука
win_sound.set_volume(0.3) #Контроль громкости звука
rem.set_volume(0.3) #Контроль громкости звука
ref.set_volume(0.3) #Контроль громкости звука
font = pygame.font.Font('textures/font.ttf', 42) #Импорт шрифта
gameover_font = pygame.font.Font('textures/font.ttf',120) #Импорт шрифта
end_score = pygame.font.Font('textures/font.ttf',70) #Импорт шрифта

def toFixed(numObj, digits=0): #команда для убирание цифр после запятой
    return f"{numObj:.{digits}f}"
def detect_collision(dx, dy, ball, rect): #Проверка коллизии
    if dx > 0: #Пока множитель больше нуля
        delta_x = ball.right - rect.left #Из координаты правой точки шара вычетается координата правой точки квадрта(строчка 10)
    else:
        delta_x = rect.right - ball.left #Из координаты правой точки квадрта(строчка 10) вычетается координата левой точки шара
    if dy > 0:
        delta_y = ball.bottom - rect.top #Из координаты нижней точки шара вычетается координата верхней точки квадрта(строчка 10)
    else:
        delta_y = rect.bottom - ball.top #Из координаты нижней точки квадрта(строчка 10) вычетается координата верхней точки шара

    if abs(delta_x - delta_y) < 10: #Если абсолютная разница координат дельт X и Y меньше 10
        dx, dy = -dx, -dy #Множители отражаются
    elif delta_x > delta_y: #Если дельта X больше дельта Y
        dy = -dy #Отражается
    elif delta_y > delta_x: #Если дельта Y больше дельта X
        dx = -dx #Отражается
    return dx, dy #Возвращает дельта X и Y
def show_score(x, y):
    score = font.render("Уничтожено " + str(scoreSum), True, (255, 255, 255))  #Показ счетчика
    Timer_Show = font.render("Время " + Timer, True, (255,255,255)) #Показ время
    if end == 1:
        sc.blit(score, (x, y)) #Координаты счета
        sc.blit(Timer_Show, (x+970, y)) #Координаты времени
def gameover():
        sc.fill((0, 0, 0))  # Заполнение черным
        gameover_text = gameover_font.render("КОНЕЦ ИГРЫ", True, (255, 255, 255))  # Экран конца игры
        sc.blit(gameover_text, (WIDTH // 4 - 60 , HEIGHT // 3 +30))  # Отображение
        flag = (Timer_score // 5) # Подсчет для проверки
        if flag == 0:
            score = 0
        else:
            score = toFixed((scoreSum * 1000/(Timer_score // 5)),0) #Подсчет финальных очков
        end_text = end_score.render("Финальный счет  "+ str(score), True, (255,255,255)) #Рендер текста
        sc.blit(end_text,(WIDTH // 4 - 70  , HEIGHT // 2 + 40)) #Вывод текста
        pygame.mixer.music.pause() #Остановка музыки
def win():
        sc.fill((0, 0, 0))  # Заполнение черным
        gameover_text = gameover_font.render("ТЫ ПОБЕДИЛ", True, (255, 255, 255))  # Экран конца игры
        sc.blit(gameover_text, (WIDTH // 4 - 70, HEIGHT // 3 + 30))  # Отображение
        score = toFixed((scoreSum * 1000 / (Timer_score // 5)),0)  #Подсчет финальных очков
        end_text = end_score.render("Финальный счет  " + str(score), True, (255, 255, 255)) #Рендер текста
        sc.blit(end_text, (WIDTH // 4 - 70, HEIGHT // 2 + 40)) #Вывод текста
        pygame.mixer.music.pause() #Остановка музыки
class Timer_thread(threading.Thread): #Новый демонический поток
    def __init__(self, interval):
        threading.Thread.__init__(self)
        self.daemon = True #Поток- демон
        self.interval = interval #Интервал

    def run(self) -> None: #Таймер
        global Timer
        Stopwatch = 0
        while True and end == 1:
            time.sleep(self.interval)
            Stopwatch += self.interval
            Timer = toFixed(Stopwatch, 0)
Timer_thread(1).start()
pygame.mixer.music.play(6)


while True: #Основной цикл
    for event in pygame.event.get(): #Проверка очереди цикла ивентов
        if event.type == pygame.QUIT: #Если ивент равен ивенту закрытия(нажата кнопка закрытия окна)
            exit() #Закрыть
    sc.blit(background, (0, 0)) #Отображение заднего фона на координатах 0 0
    [pygame.draw.rect(sc, color_list[color], block) for color, block in enumerate(block_list)] #Отрисовка массива противников
    sc.blit(Player, (PlayerX, PlayerY)) #Отрисовка игрока
    pygame.draw.circle(sc, pygame.Color('white'), ball.center, ball_radius) #Отрисовка шарика
    ball.x += ball_speed * dx * end #Измение полета шара по координате X
    ball.y += ball_speed * dy * end #Измение полета шара по координате Y
    if ball.centerx < ball_radius or ball.centerx > WIDTH - ball_radius: #Если центр по X меньше радиуса или центр по X больше ширины минус радиус
        dx = -dx #Отразить по X
        ref.play() #Запуск звука
    if ball.centery < ball_radius: #Если центр по Y меньше радиуса
        dy = -dy #Отразить по Y
        ref.play() #Запуск звука
    if ball.colliderect(paddle) and dy > 0: #Если квадраты мяча и противников пересекаются, а так же множитель Y больше 0
        dx, dy = detect_collision(dx, dy, ball, paddle) #Запуск проверки колисии и получение новых множителей
        ref.play() #Запуск звука
    hit_index = ball.collidelist(block_list) #Запись индекса противника в которого попали
    if hit_index != -1: #Если индекс не равен -1
        hit_rect = block_list.pop(hit_index) #Запись координат противника и удаление его из основного списка
        hit_color = color_list.pop(hit_index) #Запись цвета противника и удаление его из основного списка
        dx, dy = detect_collision(dx, dy, ball, hit_rect) #Запуск проверки колизии и получение новых множителей
        rem.play() #Запуск звука
        scoreSum += 1 #Добавление очков
        hit_rect.inflate_ip(ball.width * 3, ball.height * 3) #Спецэффекты
        pygame.draw.rect(sc, hit_color, hit_rect) #Спецэффекты
        fps += 2 #усложнение

    if ball.bottom > HEIGHT: #Если нижнии координаты мяча косаются нижней грани экрана
        end = 0
        Timer_score = int(Timer)
        if flag_sound == True:
            lose.play() #Запуск звука
            flag_sound = False
        gameover() #Запуск подпрограммы
    elif not len(block_list):#Если в листе нет больше противников
        end = 0
        Timer_score = int(Timer)
        if flag_sound == True:
            win_sound.play() #Запуск звука
            flag_sound = False
        win() #Запуск подпрограммы
    key = pygame.key.get_pressed() #Присвоение переменной key проверку нажатой кнопки
    if key[pygame.K_a] and paddle.left > 0: #Если нажата кнопка а и левая координата игрока больше нуля
        paddle.left -= paddle_speed #Вычесть из левой координаты скорость
        PlayerX -= paddle_speed #Сдвиг текстуры
    if key[pygame.K_d] and paddle.right < WIDTH: #Если нажата кнопка d и левая координата игрока меньше ширины
        paddle.right += paddle_speed #Сдвиг текстуры
        PlayerX += paddle_speed #Прибавить к правой координаты скорость

    show_score(20, 20) #Показ счета
    pygame.display.update() #Обновление кадра
    clock.tick(fps) #Ограничитель кадров
