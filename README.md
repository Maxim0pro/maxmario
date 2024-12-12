import pygame as pg
import random as rd


clock = pg.time.Clock()
pg.init()

display_width = 1300
display_height = 790
win = pg.display.set_mode((display_width, display_height), flags=pg.NOFRAME)
pg.display.set_caption('Mario')
pg.display.set_icon(pg.image.load('images/Details/icon.png'))

bg_image = pg.image.load('images/details/fon.png').convert_alpha()
xp_image = pg.image.load('images/details/xp.png').convert_alpha()

mushroom_images = [
    pg.image.load('images/details/mushroom1.png').convert_alpha(),
    pg.image.load('images/details/mushroom2.png').convert_alpha(),
]

cloud_images = [
    pg.image.load('images/details/cloud1.png').convert_alpha(),
    pg.image.load('images/details/cloud2.png').convert_alpha(),
    pg.image.load('images/details/cloud3.png').convert_alpha(),
]

mario_right = [
    pg.image.load('images/mario/mario1.png').convert_alpha(),
    pg.image.load('images/mario/mario2.png').convert_alpha(),
    pg.image.load('images/mario/mario3.png').convert_alpha(),
    pg.image.load('images/mario/mario4.png').convert_alpha(),
    pg.image.load('images/mario/mario5.png').convert_alpha(),
    pg.image.load('images/mario/mario6.png').convert_alpha(),
    pg.image.load('images/mario/mario7.png').convert_alpha(),
]

mario_left = [
    pg.image.load('images/mario2/mario1.png').convert_alpha(),
    pg.image.load('images/mario2/mario2.png').convert_alpha(),
    pg.image.load('images/mario2/mario3.png').convert_alpha(),
    pg.image.load('images/mario2/mario4.png').convert_alpha(),
    pg.image.load('images/mario2/mario5.png').convert_alpha(),
    pg.image.load('images/mario2/mario6.png').convert_alpha(),
    pg.image.load('images/mario2/mario7.png').convert_alpha(),
]


# Шрифты, размеры и текст для паузы
font = pg.font.Font('images/Details/text.ttf', 70)
paused_text = font.render('The game is suspended', True, (0, 32, 255))
time_font = pg.font.Font('images/details/text.ttf', 24)


# Функция для вывода текста на эран
def print_text(message, x, y, font_color=(0, 0, 0), font_type='images/details/text.ttf', font_size=50):
    font_type = pg.font.Font(font_type, font_size)
    text = font_type.render(message, True, font_color)
    win.blit(text, (x, y))


# Функция для вывода текста на экран
def draw_text(surface, text, color, x, y):
    textobj = font.render(text, 1, color)
    textrect = textobj.get_rect()
    textrect.topleft = (x, y)
    surface.blit(textobj, textrect)


# Функция для отображения текста на экран
def draw_paused_win():
    win.blit(paused_text, paused_text.get_rect(center=(600, 350)))
    pg.display.flip()


anim_count = 0
mario_speed = 10
mario_x, mario_y = 10, 192
mushroom_x, mushroom_y = 300, 201
jump_count = 6
border_left = 0
border_right = display_height
border_top = 0
border_bottom = 50
mushroom_direction = 1
mushroom_speed = 2
mushroom_frame = 0
animation_speed = 30
star_time = 64
current_time = star_time
fall_border_x = 1210
floor_y = 455


text = 'Game Over'
color = (152, 0, 2)
text_x, text_y = 500, 310


bg_sound = pg.mixer.Sound('sounds/background.mp3')
bg_sound.play(loops=-1)
jump_sound = pg.mixer.Sound('sounds/jump.mp3')
loss_sound = pg.mixer.Sound('sounds/gameover.mp3')
pause_sound = pg.mixer.Sound('sounds/pause.mp3')
button_sound = pg.mixer.Sound('sounds/pause.mp3')


# Класс кнопки с методом для рисования и обработки кликов
class Button:

    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.inactive_color = (255, 0, 0)
        self.active_color = (128, 128, 128)

    def draw(self, x, y, message, action=None):
        mouse = pg.mouse.get_pos()
        click = pg.mouse.get_pressed()
        if x < mouse[0] < x + self.width and y < mouse[1] < y + self.height:
            pg.draw.rect(win, self.active_color, (x, y, self.width, self.height))
            if click[0] == 1:
                pg.mixer.Sound.play(button_sound)
                pg.time.delay(300)
                if action is not None:
                    action()
        else:
            pg.draw.rect(win, self.inactive_color, (x, y, self.width, self.height))
        print_text(message, x + 10, x + 10)


game_paused = False
jump = False
game_over = False
running = True


while running:

    win.blit(bg_image, (0, 0))
    win.blit(mushroom_images[1], (840, 464))
    win.blit(mushroom_images[1], (400, 728))
    xp_positions = [(20, 20), (55, 20), (90, 20)]
    for pos in xp_positions:
        win.blit(xp_image, pos)

    player_rect = mario_left[0].get_rect(topleft=(mario_x, mario_y))
    mushroom_rect = mushroom_images[0].get_rect(topleft=(mushroom_x, mushroom_y))

# Проверка столкновений между Марио и грибом
    if player_rect.colliderect(mushroom_rect):
        draw_text(win, text, color, text_x, text_y)

    keys = pg.key.get_pressed()
    if keys[pg.K_a]:
        win.blit(mario_left[anim_count], (mario_x, mario_y))
    else:
        win.blit(mario_right[anim_count], (mario_x, mario_y))

    if keys[pg.K_a] and mario_x > 1:
        mario_x -= mario_speed
    elif keys[pg.K_d] and mario_x < 1280:
        mario_x += mario_speed

    if not jump:
        if keys[pg.K_SPACE]:
            jump = True
            jump_sound.play()
    else:
        if jump_count >= -6:
            if jump_count > 0:
                mario_y -= (jump_count ** 2) / 2
            else:
                mario_y += (jump_count ** 2) / 2
            jump_count -= 1
        else:
            jump = False
            jump_count = 6

    if anim_count == 6:
        anim_count = 0
    elif keys[pg.K_a] or keys[pg.K_d]:
        anim_count += 1

    button = Button(300, 100)
    button.draw(400, 400, 'wow')

    # Обновляем координаты гриба
    mushroom_x += mushroom_speed * mushroom_direction

    # Проверяем границы
    if mushroom_x <= border_left or mushroom_x >= border_right - mushroom_images[0].get_width():
        mushroom_direction *= -1

    # Анимация гриба
    mushroom_frame += 1
    if mushroom_frame >= animation_speed:
        mushroom_frame = 0

    # Выбираем текущее изображение гриба
    current_mushroom_image = mushroom_images[mushroom_frame // (animation_speed // len(mushroom_images))]

    # Отрисовка гриба
    win.blit(current_mushroom_image, (mushroom_x, mushroom_y))

    # Обновляем таймер уменьшая оставшееся время на велечину прошедшего времени с момента последнего обновления
    dt = clock.tick(140) / 100
    current_time -= dt
    if current_time <= 0:
        current_time = 0

    time_text = time_font.render(f'Time: {int(current_time)}', True, (0, 0, 0))
    win.blit(time_text, (600, 20))

    if mario_x >= fall_border_x:
        if mario_y < floor_y:
            mario_y += 10
        else:
            mario_y = floor_y

    for event in pg.event.get():
        if event.type == pg.QUIT:
            running = False
        elif event.type == pg.KEYDOWN:
            if event.key == pg.K_ESCAPE:
                pause_sound.play()
                game_paused = not game_paused

    if game_paused and not game_over:
        draw_paused_win()

    if not game_over and player_rect.colliderect(mushroom_rect):
        loss_sound.play()
        draw_text(win, text, color, text_x, text_y)
        game_over = True

    clock.tick(15)
    pg.display.update()
