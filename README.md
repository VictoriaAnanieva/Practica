import pygame
from random import randrange as rnd
import time
import threading

pygame.init() 

WIDTH, HEIGHT = 1200, 800 
fps = 60 
paddle_w, paddle_h, paddle_speed  = 330, 35, 15  
ball_radius, ball_speed  = 20, 6 
ball_rect = int(ball_radius * 2 ** 0.5) 
dx, dy = 1, -1 
PlayerX = WIDTH // 2 - paddle_w // 2 
PlayerY = HEIGHT - paddle_h - 10 
paddle = pygame.Rect(WIDTH // 2 - paddle_w // 2, HEIGHT - paddle_h - 10, paddle_w, paddle_h) 
ball = pygame.Rect(rnd(ball_rect, WIDTH - ball_rect), HEIGHT // 2, ball_rect, ball_rect) 
block_list = [pygame.Rect(10 + 120 * i, 10 + 70 * j, 100, 50) for i in range(10) for j in range(4)] 
color_list = [(rnd(0,40), rnd(0,40), rnd(30,256)) for i in range(10) for j in range(4)] 
data_lock= threading.Lock() 

Timer = "0" 
Timer_Show = 0 
score, scoreSum  = 0, 0 
end = 1 
flag_sound = True 
flag = 0 

sc = pygame.display.set_mode((WIDTH, HEIGHT)) 
clock = pygame.time.Clock()

background = pygame.transform.scale (pygame.image.load('textures/background.jpg'), (1200,800)) 
Player = pygame.transform.scale(pygame.image.load('textures/Player.png'), (330, 35)) 
pygame.display.set_caption("Защита леса") 
pygame.display.set_icon(pygame.image.load('textures/logo.bmp')) 
ref = pygame.mixer.Sound('textures/reflection.mp3') 
rem = pygame.mixer.Sound('textures/remove.mp3') 
lose = pygame.mixer.Sound('textures/lose.mp3') 
win_sound = pygame.mixer.Sound('textures/win.mp3') 
pygame.mixer.music.load('textures/music.mp3') 
pygame.mixer.music.set_volume(0.2) 
lose.set_volume(0.3) 
win_sound.set_volume(0.3) 
rem.set_volume(0.3) 
ref.set_volume(0.3) 
font = pygame.font.Font('textures/font.ttf', 42) 
gameover_font = pygame.font.Font('textures/font.ttf',120) 
end_score = pygame.font.Font('textures/font.ttf',70) 

def toFixed(numObj, digits=0): 
    return f"{numObj:.{digits}f}"
def detect_collision(dx, dy, ball, rect): 
    if dx > 0: 
        delta_x = ball.right - rect.left 
    else:
        delta_x = rect.right - ball.left 
    if dy > 0:
        delta_y = ball.bottom - rect.top 
    else:
        delta_y = rect.bottom - ball.top 
    if abs(delta_x - delta_y) < 10: 
        dx, dy = -dx, -dy 
    elif delta_x > delta_y: 
        dy = -dy 
    elif delta_y > delta_x: 
        dx = -dx 
    return dx, dy 
def show_score(x, y):
    score = font.render("Уничтожено " + str(scoreSum), True, (255, 255, 255)) 
    Timer_Show = font.render("Время " + Timer, True, (255,255,255)) 
    if end == 1:
        sc.blit(score, (x, y)) 
        sc.blit(Timer_Show, (x+970, y)) 
def gameover():
        sc.fill((0, 0, 0))  # 
        gameover_text = gameover_font.render("КОНЕЦ ИГРЫ", True, (255, 255, 255))  # 
        sc.blit(gameover_text, (WIDTH // 4 - 60 , HEIGHT // 3 +30))  
        flag = (Timer_score // 5) 
        if flag == 0:
            score = 0
        else:
            score = toFixed((scoreSum * 1000/(Timer_score // 5)),0) 
        end_text = end_score.render("Финальный счет  "+ str(score), True, (255,255,255)) 
        sc.blit(end_text,(WIDTH // 4 - 70  , HEIGHT // 2 + 40)) 
        pygame.mixer.music.pause() 
def win():
        sc.fill((0, 0, 0))  
        gameover_text = gameover_font.render("ТЫ ПОБЕДИЛ", True, (255, 255, 255))  
        sc.blit(gameover_text, (WIDTH // 4 - 70, HEIGHT // 3 + 30))  
        score = toFixed((scoreSum * 1000 / (Timer_score // 5)),0)  
        end_text = end_score.render("Финальный счет  " + str(score), True, (255, 255, 255))
        sc.blit(end_text, (WIDTH // 4 - 70, HEIGHT // 2 + 40)) 
        pygame.mixer.music.pause() 
class Timer_thread(threading.Thread):
    def __init__(self, interval):
        threading.Thread.__init__(self)
        self.daemon = True #Поток- демон
        self.interval = interval 

    def run(self) -> None: 
        global Timer
        Stopwatch = 0
        while True and end == 1:
            time.sleep(self.interval)
            Stopwatch += self.interval
            Timer = toFixed(Stopwatch, 0)
Timer_thread(1).start()
pygame.mixer.music.play(6)


while True: #Основной цикл
    for event in pygame.event.get(): 
        if event.type == pygame.QUIT: 
            exit() #Закрыть
    sc.blit(background, (0, 0)) 
    [pygame.draw.rect(sc, color_list[color], block) for color, block in enumerate(block_list)] 
    sc.blit(Player, (PlayerX, PlayerY)) 
    pygame.draw.circle(sc, pygame.Color('white'), ball.center, ball_radius) 
    ball.x += ball_speed * dx * end 
    ball.y += ball_speed * dy * end 
    if ball.centerx < ball_radius or ball.centerx > WIDTH - ball_radius: 
        dx = -dx #Отразить по X
        ref.play()
    if ball.centery < ball_radius: 
        dy = -dy #Отразить по Y
        ref.play() 
    if ball.colliderect(paddle) and dy > 0: #Если квадраты мяча и противников пересекаются, а так же множитель Y больше 0
        dx, dy = detect_collision(dx, dy, ball, paddle) #Запуск проверки колисии и получение новых множителей
        ref.play() 
    hit_index = ball.collidelist(block_list) #Запись индекса противника в которого попали
    if hit_index != -1: 
        hit_rect = block_list.pop(hit_index) 
        hit_color = color_list.pop(hit_index) 
        dx, dy = detect_collision(dx, dy, ball, hit_rect) 
        rem.play() 
        scoreSum += 1 
        hit_rect.inflate_ip(ball.width * 3, ball.height * 3) 
        pygame.draw.rect(sc, hit_color, hit_rect) 
        fps += 2 

    if ball.bottom > HEIGHT:
        end = 0
        Timer_score = int(Timer)
        if flag_sound == True:
            lose.play() 
            flag_sound = False
        gameover() #Запуск подпрограммы
    elif not len(block_list):
        end = 0
        Timer_score = int(Timer)
        if flag_sound == True:
            win_sound.play() 
            flag_sound = False
        win() 
    key = pygame.key.get_pressed() 
    if key[pygame.K_a] and paddle.left > 0: 
        paddle.left -= paddle_speed 
        PlayerX -= paddle_speed 
    if key[pygame.K_d] and paddle.right < WIDTH: 
        paddle.right += paddle_speed 
        PlayerX += paddle_speed 

    show_score(20, 20) 
    pygame.display.update() 
    clock.tick(fps) #
