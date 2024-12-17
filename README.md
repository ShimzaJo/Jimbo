import os
import sys
import pygame
import random
from pygame import *

pygame.init()
screenDisplay = (width_screen, height_screen) = (600, 400)
FPS = 60
gravity = 0.6

black_color = (0,0,0)
white_color = (255,255,255)
backgroundColor = (235, 235, 235)

highest_scores = 0

screenDisplay = pygame.display.set_mode(screenDisplay)
timerClock = pygame.time.Clock()
pygame.display.set_caption("Dino Run - CopyAssignment ")

soundOnJump = pygame.mixer.Sound('resources/jump.wav')
soundOnDie = pygame.mixer.Sound('resources/die.wav')
soundOnCheckpoint = pygame.mixer.Sound('resources/checkPoint.wav')

def load_image(
    name,
    sx=-1,
    sy=-1,
    colorkey=None,
    ):

    fullname = os.path.join('resources', name)
    img = pygame.image.load(fullname)
    img = img.convert()
    if colorkey is not None:
        if colorkey == -1:
            colorkey = img.get_at((0, 0))
        img.set_colorkey(colorkey, RLEACCEL)

    if sx != -1 or sy != -1:
        img = pygame.transform.scale(img, (sx, sy))

    return (img, img.get_rect())

def load_sprite_sheet(
        s_name,
        namex,
        namey,
        scx = -1,
        scy = -1,
        c_key = None,
        ):
    fullname = os.path.join('resources', s_name)
    sh = pygame.image.load(fullname)
    sh = sh.convert()

    sh_rect = sh.get_rect()

    sprites = []

    sx = sh_rect.width/ namex
    sy = sh_rect.height/ namey

    for i in range(0, namey):
        for j in range(0, namex):
            rect = pygame.Rect((j*sx,i*sy,sx,sy))
            img = pygame.Surface(rect.size)
            img = img.convert()
            img.blit(sh,(0,0),rect)

            if c_key is not None:
                if c_key == -1:
                    c_key = img.get_at((0, 0))
                img.set_colorkey(c_key, RLEACCEL)

            if scx != -1 or scy != -1:
                img = pygame.transform.scale(img, (scx, scy))

            sprites.append(img)

    sprite_rect = sprites[0].get_rect()

    return sprites,sprite_rect

def gameover_display_message(rbtn_image, gmo_image):
    rbtn_rect = rbtn_image.get_rect()
    rbtn_rect.centerx = width_screen / 2
    rbtn_rect.top = height_screen * 0.52

    gmo_rect = gmo_image.get_rect()
    gmo_rect.centerx = width_screen / 2
    gmo_rect.centery = height_screen * 0.35

    screenDisplay.blit(rbtn_image, rbtn_rect)
    screenDisplay.blit(gmo_image, gmo_rect)

def extractDigits(num):
    if num > -1:
        d = []
        i = 0
        while(num / 10 != 0):
            d.append(num % 10)
            num = int(num / 10)

        d.append(num % 10)
        for i in range(len(d),5):
            d.append(0)
        d.reverse()
        return d

class Dino():
    def __init__(self, sx=-1, sy=-1):
        self.imgs, self.rect = load_sprite_sheet('dino.png', 5, 1, sx, sy, -1)
        self.imgs1, self.rect1 = load_sprite_sheet('dino_ducking.png', 2, 1, 59, sy, -1)
        self.rect.bottom = int(0.98 * height_screen)
        self.rect.left = width_screen / 15
        self.image = self.imgs[0]
        self.index = 0
        self.counter = 0
        self.score = 0
        self.jumping = False
        self.dead = False
        self.ducking = False
        self.blinking = False
        self.movement = [0,0]
        self.jumpSpeed = 11.5

        self.stand_position_width = self.rect.width
        self.duck_position_width = self.rect1.width

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def checkbounds(self):
        if self.rect.bottom > int(0.98 * height_screen):
            self.rect.bottom = int(0.98 * height_screen)
            self.jumping = False

    def update(self):
        if self.jumping:
            self.movement[1] = self.movement[1] + gravity

        if self.jumping:
            self.index = 0
        elif self.blinking:
            if self.index == 0:
                if self.counter % 400 == 399:
                    self.index = (self.index + 1)%2
            else:
                if self.counter % 20 == 19:
                    self.index = (self.index + 1)%2

        elif self.ducking:
            if self.counter % 5 == 0:
                self.index = (self.index + 1)%2
        else:
            if self.counter % 5 == 0:
                self.index = (self.index + 1)%2 + 2

        if self.dead:
           self.index = 4

        if not self.ducking:
            self.image = self.imgs[self.index]
            self.rect.width = self.stand_position_width
        else:
            self.image = self.imgs1[(self.index) % 2]
            self.rect.width = self.duck_position_width

        self.rect = self.rect.move(self.movement)
        self.checkbounds()

        if not self.dead and self.counter % 7 == 6 and self.blinking == False:
            self.score += 1
            if self.score % 100 == 0 and self.score != 0:
                if pygame.mixer.get_init() != None:
                    soundOnCheckpoint.play()

        self.counter = (self.counter + 1)

class birds(pygame.sprite.Sprite):
    def __init__(self, speed=5, sx=-1, sy=-1):
        pygame.sprite.Sprite.__init__(self,self.containers)
        self.imgs, self.rect = load_sprite_sheet('birds.png', 2, 1, sx, sy, -1)
        self.birds_height = [height_screen * 0.82, height_screen * 0.75, height_screen * 0.60]
        self.rect.centery = self.birds_height[random.randrange(0, 3)]
        self.rect.left = width_screen + self.rect.width
        self.image = self.imgs[0]
        self.movement = [-1*speed,0]
        self.index = 0
        self.counter = 0

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def update(self):
        if self.counter % 10 == 0:
            self.index = (self.index+1)%2
        self.image = self.imgs[self.index]
        self.rect = self.rect.move(self.movement)
        self.counter = (self.counter + 1)
        if self.rect.right < 0:
            self.kill()

class Ground():
    def __init__(self,speed=-5):
        self.image,self.rect = load_image('ground.png',-1,-1,-1)
        self.image1,self.rect1 = load_image('ground.png',-1,-1,-1)
        self.rect.bottom = height_screen
        self.rect1.bottom = height_screen
        self.rect1.left = self.rect.right
        self.speed = speed

    def draw(self):
        screenDisplay.blit(self.image, self.rect)
        screenDisplay.blit(self.image1, self.rect1)

    def update(self):
        self.rect.left += self.speed
        self.rect1.left += self.speed

        if self.rect.right < 0:
            self.rect.left = self.rect1.right

        if self.rect1.right < 0:
            self.rect1.left = self.rect.right

class Scoreboard():
    def __init__(self,x=-1,y=-1):
        self.score = 0
        self.scre_img, self.screrect = load_sprite_sheet('numbers.png', 12, 1, 11, int(11 * 6 / 5), -1)
        self.image = pygame.Surface((55,int(11*6/5)))
        self.rect = self.image.get_rect()
        if x == -1:
            self.rect.left = width_screen * 0.89
        else:
            self.rect.left = x
        if y == -1:
            self.rect.top = height_screen * 0.1
        else:
            self.rect.top = y

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def update(self,score):
        score_digits = extractDigits(score)
        self.image.fill(backgroundColor)
        for s in score_digits:
            self.image.blit(self.scre_img[s], self.screrect)
            self.screrect.left += self.screrect.width
        self.screrect.left = 0

def introduction_screen():
    ado_dino = Dino(44,47)
    ado_dino.blinking = True
    starting_game = False

    t_ground,t_ground_rect = load_sprite_sheet('ground.png',15,1,-1,-1,-1)
    t_ground_rect.left = width_screen / 20
    t_ground_rect.bottom = height_screen

    logo,l_rect = load_image('logo.png',300,140,-1)
    l_rect.centerx = width_screen * 0.6
    l_rect.centery = height_screen * 0.6
    while not starting_game:
        if pygame.display.get_surface() == None:
            print("Couldn't load display surface")
            return True
        else:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    return True
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_SPACE or event.key == pygame.K_UP:
                        ado_dino.jumping = True
                        ado_dino.blinking = False
                        ado_dino.movement[1] = -1*ado_dino.jumpSpeed

        ado_dino.update()

        if pygame.display.get_surface() != None:
            screenDisplay.fill(backgroundColor)
            screenDisplay.blit(t_ground[0], t_ground_rect)
            if ado_dino.blinking:
                screenDisplay.blit(logo, l_rect)
            ado_dino.draw()

            pygame.display.update()

        timerClock.tick(FPS)
        if ado_dino.jumping == False and ado_dino.blinking == False:
            starting_game = True



import os
import sys
import pygame
import random
from pygame import *

pygame.init()
screenDisplay = (width_screen, height_screen) = (600, 400)
FPS = 60
gravity = 0.6

black_color = (0,0,0)
white_color = (255,255,255)
backgroundColor = (235, 235, 235)

highest_scores = 0

screenDisplay = pygame.display.set_mode(screenDisplay)
timerClock = pygame.time.Clock()
pygame.display.set_caption("Dino Run - CopyAssignment ")

soundOnJump = pygame.mixer.Sound('resources/jump.wav')
soundOnDie = pygame.mixer.Sound('resources/die.wav')
soundOnCheckpoint = pygame.mixer.Sound('resources/checkPoint.wav')

def load_image(
    name,
    sx=-1,
    sy=-1,
    colorkey=None,
    ):

    fullname = os.path.join('resources', name)
    img = pygame.image.load(fullname)
    img = img.convert()
    if colorkey is not None:
        if colorkey == -1:
            colorkey = img.get_at((0, 0))
        img.set_colorkey(colorkey, RLEACCEL)

    if sx != -1 or sy != -1:
        img = pygame.transform.scale(img, (sx, sy))

    return (img, img.get_rect())

def load_sprite_sheet(
        s_name,
        namex,
        namey,
        scx = -1,
        scy = -1,
        c_key = None,
        ):
    fullname = os.path.join('resources', s_name)
    sh = pygame.image.load(fullname)
    sh = sh.convert()

    sh_rect = sh.get_rect()

    sprites = []

    sx = sh_rect.width/ namex
    sy = sh_rect.height/ namey

    for i in range(0, namey):
        for j in range(0, namex):
            rect = pygame.Rect((j*sx,i*sy,sx,sy))
            img = pygame.Surface(rect.size)
            img = img.convert()
            img.blit(sh,(0,0),rect)

            if c_key is not None:
                if c_key == -1:
                    c_key = img.get_at((0, 0))
                img.set_colorkey(c_key, RLEACCEL)

            if scx != -1 or scy != -1:
                img = pygame.transform.scale(img, (scx, scy))

            sprites.append(img)

    sprite_rect = sprites[0].get_rect()

    return sprites,sprite_rect

def gameover_display_message(rbtn_image, gmo_image):
    rbtn_rect = rbtn_image.get_rect()
    rbtn_rect.centerx = width_screen / 2
    rbtn_rect.top = height_screen * 0.52

    gmo_rect = gmo_image.get_rect()
    gmo_rect.centerx = width_screen / 2
    gmo_rect.centery = height_screen * 0.35

    screenDisplay.blit(rbtn_image, rbtn_rect)
    screenDisplay.blit(gmo_image, gmo_rect)

def extractDigits(num):
    if num > -1:
        d = []
        i = 0
        while(num / 10 != 0):
            d.append(num % 10)
            num = int(num / 10)

        d.append(num % 10)
        for i in range(len(d),5):
            d.append(0)
        d.reverse()
        return d

class Dino():
    def __init__(self, sx=-1, sy=-1):
        self.imgs, self.rect = load_sprite_sheet('dino.png', 5, 1, sx, sy, -1)
        self.imgs1, self.rect1 = load_sprite_sheet('dino_ducking.png', 2, 1, 59, sy, -1)
        self.rect.bottom = int(0.98 * height_screen)
        self.rect.left = width_screen / 15
        self.image = self.imgs[0]
        self.index = 0
        self.counter = 0
        self.score = 0
        self.jumping = False
        self.dead = False
        self.ducking = False
        self.blinking = False
        self.movement = [0,0]
        self.jumpSpeed = 11.5

        self.stand_position_width = self.rect.width
        self.duck_position_width = self.rect1.width

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def checkbounds(self):
        if self.rect.bottom > int(0.98 * height_screen):
            self.rect.bottom = int(0.98 * height_screen)
            self.jumping = False

    def update(self):
        if self.jumping:
            self.movement[1] = self.movement[1] + gravity

        if self.jumping:
            self.index = 0
        elif self.blinking:
            if self.index == 0:
                if self.counter % 400 == 399:
                    self.index = (self.index + 1)%2
            else:
                if self.counter % 20 == 19:
                    self.index = (self.index + 1)%2

        elif self.ducking:
            if self.counter % 5 == 0:
                self.index = (self.index + 1)%2
        else:
            if self.counter % 5 == 0:
                self.index = (self.index + 1)%2 + 2

        if self.dead:
           self.index = 4

        if not self.ducking:
            self.image = self.imgs[self.index]
            self.rect.width = self.stand_position_width
        else:
            self.image = self.imgs1[(self.index) % 2]
            self.rect.width = self.duck_position_width

        self.rect = self.rect.move(self.movement)
        self.checkbounds()

        if not self.dead and self.counter % 7 == 6 and self.blinking == False:
            self.score += 1
            if self.score % 100 == 0 and self.score != 0:
                if pygame.mixer.get_init() != None:
                    soundOnCheckpoint.play()

        self.counter = (self.counter + 1)

class birds(pygame.sprite.Sprite):
    def __init__(self, speed=5, sx=-1, sy=-1):
        pygame.sprite.Sprite.__init__(self,self.containers)
        self.imgs, self.rect = load_sprite_sheet('birds.png', 2, 1, sx, sy, -1)
        self.birds_height = [height_screen * 0.82, height_screen * 0.75, height_screen * 0.60]
        self.rect.centery = self.birds_height[random.randrange(0, 3)]
        self.rect.left = width_screen + self.rect.width
        self.image = self.imgs[0]
        self.movement = [-1*speed,0]
        self.index = 0
        self.counter = 0

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def update(self):
        if self.counter % 10 == 0:
            self.index = (self.index+1)%2
        self.image = self.imgs[self.index]
        self.rect = self.rect.move(self.movement)
        self.counter = (self.counter + 1)
        if self.rect.right < 0:
            self.kill()

class Ground():
    def __init__(self,speed=-5):
        self.image,self.rect = load_image('ground.png',-1,-1,-1)
        self.image1,self.rect1 = load_image('ground.png',-1,-1,-1)
        self.rect.bottom = height_screen
        self.rect1.bottom = height_screen
        self.rect1.left = self.rect.right
        self.speed = speed

    def draw(self):
        screenDisplay.blit(self.image, self.rect)
        screenDisplay.blit(self.image1, self.rect1)

    def update(self):
        self.rect.left += self.speed
        self.rect1.left += self.speed

        if self.rect.right < 0:
            self.rect.left = self.rect1.right

        if self.rect1.right < 0:
            self.rect1.left = self.rect.right

class Scoreboard():
    def __init__(self,x=-1,y=-1):
        self.score = 0
        self.scre_img, self.screrect = load_sprite_sheet('numbers.png', 12, 1, 11, int(11 * 6 / 5), -1)
        self.image = pygame.Surface((55,int(11*6/5)))
        self.rect = self.image.get_rect()
        if x == -1:
            self.rect.left = width_screen * 0.89
        else:
            self.rect.left = x
        if y == -1:
            self.rect.top = height_screen * 0.1
        else:
            self.rect.top = y

    def draw(self):
        screenDisplay.blit(self.image, self.rect)

    def update(self,score):
        score_digits = extractDigits(score)
        self.image.fill(backgroundColor)
        for s in score_digits:
            self.image.blit(self.scre_img[s], self.screrect)
            self.screrect.left += self.screrect.width
        self.screrect.left = 0

def introduction_screen():
    ado_dino = Dino(44,47)
    ado_dino.blinking = True
    starting_game = False

    t_ground,t_ground_rect = load_sprite_sheet('ground.png',15,1,-1,-1,-1)
    t_ground_rect.left = width_screen / 20
    t_ground_rect.bottom = height_screen

    logo,l_rect = load_image('logo.png',300,140,-1)
    l_rect.centerx = width_screen * 0.6
    l_rect.centery = height_screen * 0.6
    while not starting_game:
        if pygame.display.get_surface() == None:
            print("Couldn't load display surface")
            return True
        else:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    return True
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_SPACE or event.key == pygame.K_UP:
                        ado_dino.jumping = True
                        ado_dino.blinking = False
                        ado_dino.movement[1] = -1*ado_dino.jumpSpeed

        ado_dino.update()

        if pygame.display.get_surface() != None:
            screenDisplay.fill(backgroundColor)
            screenDisplay.blit(t_ground[0], t_ground_rect)
            if ado_dino.blinking:
                screenDisplay.blit(logo, l_rect)
            ado_dino.draw()

            pygame.display.update()

        timerClock.tick(FPS)
        if ado_dino.jumping == False and ado_dino.blinking == False:
            starting_game = True



