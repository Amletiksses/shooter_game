from pygame import *
from random import randint
from time import time as timer
# Создаем окно
win_width = 900
win_height = 700
window = display.set_mode((win_width,win_height))
display.set_caption("Space shooter")

mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
shot = mixer.Sound('fire.ogg')
# Создаем фон
background = image.load('galaxy.jpg')
background = transform.scale(background, (win_width, win_height))
# Создаем объект часы для задания кадров
clock = time.Clock()
FPS = 60
lost = 0
score = 0
life = 3
max_lost = 10 
goal = 20
font.init()
font_game = font.SysFont('Arial', 36)
font_finish = font.SysFont('Arial', 80)
#Создаем общий класс для спрайтов
class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, 
                player_width, player_height, player_speed):
        sprite.Sprite.__init__(self)
        self.image = transform.scale(image.load(player_image), 
                                        (player_width, player_height))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update(self):
        key_pressed = key.get_pressed()
        if key_pressed[K_RIGHT] and self.rect.x < win_width - 85:
            self.rect.x += self.speed
        if key_pressed[K_LEFT] and self.rect.x > 5:
            self.rect.x -= self.speed
    def fire(self):
        bullet = Bullet('bullet.png', self.rect.centerx, self.rect.top, 15, 20, -15)
        bullets.add(bullet)

class Enemy(GameSprite):
    def update(self):
        self.rect.y += self.speed
        global lost
        if self.rect.y > win_width:
            self.rect.y = -50
            self.rect.x = randint(0, win_width - 80)
            lost += 1

class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()


class Background():
    def __init__(self, filename, win_w, win_h):
        self.image = image.load(filename)
        self.image = transform.scale(self.image, (win_w, win_h))
        self.rect_1 = self.image.get_rect()
        self.rect_1.x = 0 
        self.rect_1.y = 0 
        self.rect_2 = self.image.get_rect()
        self.rect_2.x = 0 
        self.rect_2.y = -win_h
        self.moving = 3 
        self.height = win_h
        self.width = win_w
    def update(self):
        self.rect_1.y += self.moving
        self.rect_2.y += self.moving
        if self.rect_1.y > self.height:
            self.rect_1.y = -self.height
        if self.rect_2.y > self.height:
            self.rect_2.y = -self.height
    def draw(self):
        window.blit(self.image, (self.rect_1.x, self.rect_1.y))
        window.blit(self.image, (self.rect_2.x, self.rect_2.y))

# Загружаем персонажей
back = Background('galaxy.jpg', win_width,win_height)
ship = Player('rocket.png', win_width // 2, win_height - 120, 80, 120, 10)
monsters = sprite.Group()
for i in range(1,6):
    monster = Enemy('ufo.png', randint(0, win_width - 80), -70, 80, 50, randint(1,6))
    monsters.add(monster)
asteroids = sprite.Group()
for i in range(1,3):
    asteroid = Enemy('asteroid.png', randint(100, win_width- 200), -50, 80, 50, randint(3,6))
    asteroids.add(asteroid)

bullets = sprite.Group()


# Создаем игровой цикл
run = True
finish = False
rel_time = False
num_fire = 0

while run:
    # Проверка событий на выход
    for e in event.get():
        if e.type == QUIT:
            run = False
        if e.type == KEYDOWN:
            if e.key == K_SPACE:
                if num_fire < 5 and rel_time == False:
                    num_fire += 1 
                    shot.play()
                    ship.fire()
                if num_fire >= 5 and rel_time == False:
                    last_time = timer()
                    rel_time = True
                    
    if not finish:
        back.update()
        back.draw()
        ship.update()
        ship.reset()
        monsters.update()
        monsters.draw(window)
        bullets.update()
        bullets.draw(window)
        asteroids.update()
        asteroids.draw(window)

        if rel_time == True:
            now_time = timer()

            if now_time - last_time < 3:
                reload = font_game.render('Перезарядка ждите...', True, (150,0,0))
                window.blit(reload, (340,630))
            else:
                num_fire = 0 
                rel_time = False

        text_score = font_game.render("Счет:" + str(score), True, (255, 255, 255))
        window.blit(text_score, (10,10))
        text_lost = font_game.render("Пропущено:" + str(lost), True, (255, 255, 255))
        window.blit(text_lost, (10,40))

        if life == 3:
            life_color = (0, 150, 0)
        if life == 2:
            life_color = (150, 150, 0)
        if life  == 1:
            life_color = (150,0,0)
        
        text_life = font_game.render(str(life), True, life_color)
        window.blit(text_life, (win_width - 50,10))

        if sprite.spritecollide(ship, monsters, False) or sprite.spritecollide(ship, asteroids, False):
            sprite.spritecollide(ship, monsters, True)
            sprite.spritecollide(ship, asteroids, True)
            life = life - 1

        if life == 0 or lost >= max_lost:  
            finish = True
            text_lose = font_finish.render("Поражение!", True, (80,0,0))
            window.blit(text_lose, (300,300))

        collides = sprite.groupcollide(monsters, bullets, True, True)
        for c in collides:
            score += 1
            monster = Enemy('ufo.png', randint(0, win_width - 80), -70, 80, 50, randint(1,6))
            monsters.add(monster)

        if score >= goal:
            finish = True
            text_win = font_finish.render("Победа", True, (255,255,200))
            window.blit(text_win, (300,300))
        # Обновление экрана 
        display.update()
        clock.tick(FPS)  
    else:
        finish = False
        score = 0
        lost = 0 
        life = 3
        num_fire = 0  
        for bullet in bullets:
            bullet.kill()
        for monster in monsters:
            monster.kill()
        for asteroid in asteroids:
            asteroid.kill()
        
        time.delay(3000)
        for i in range(1,6):
            monster = Enemy('ufo.png', randint(0, win_width - 80), -70, 80, 50, randint(1,6))
            monsters.add(monster)
            
        
        for i in range(1,3):
            asteroid = Enemy('asteroid.png', randint(100, win_width- 200), -50, 80, 50, randint(3,6))
            asteroids.add(asteroid)
    clock.tick(FPS)




