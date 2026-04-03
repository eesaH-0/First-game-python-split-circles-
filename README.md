import pygame
import math

pygame.init()

# --- CONFIG ---
WIDTH, HEIGHT = 800, 600
FPS = 60

BIG_RADIUS = 40
SMALL_RADIUS = 20
SPEED = 5

BLUE = (40, 140, 255)
RED = (255, 60, 60)
WHITE = (255, 255, 255)
BG = (250, 250, 250)

# --- SETUP ---
win = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("divide circles")
clock = pygame.time.Clock()
font_cache = {}


def get_font(size):
    if size not in font_cache:
        font_cache[size] = pygame.font.SysFont(None, size)
    return font_cache[size]


def draw_text(text, x, y, size=30, center=False):
    font = get_font(size)
    surf = font.render(text, True, (0, 0, 0))
    rect = surf.get_rect()
    if center:
        rect.center = (x, y)
    else:
        rect.topleft = (x, y)
    win.blit(surf, rect)


class Circle:
    def __init__(self, x, y, radius=BIG_RADIUS, color=BLUE):
        self.x = x
        self.y = y
        self.r = radius
        self.color = color

    def move(self, keys, controls):
        if keys[controls["up"]]:
            self.y -= SPEED
        if keys[controls["down"]]:
            self.y += SPEED
        if keys[controls["left"]]:
            self.x -= SPEED
        if keys[controls["right"]]:
            self.x += SPEED

        self.x = max(self.r, min(self.x, WIDTH - self.r))
        self.y = max(self.r, min(self.y, HEIGHT - self.r))

    def draw(self, label=None):
        pygame.draw.circle(win, self.color, (int(self.x), int(self.y)), self.r)

        if label is not None:
            font = get_font(int(self.r * 0.9))
            text = font.render(str(label), True, WHITE)
            win.blit(text, text.get_rect(center=(self.x, self.y)))


def start_screen():
    blink = 0

    while True:
        win.fill((180, 210, 255))

        draw_text("Split / Merge Circles", WIDTH // 2, 140, 60, True)
        draw_text("P1: WASD", WIDTH // 2, 260, 30, True)
        draw_text("P2: Arrow Keys", WIDTH // 2, 300, 30, True)
        draw_text("Space = Split", WIDTH // 2, 340, 30, True)

        blink += 1
        if (blink // 30) % 2 == 0:
            draw_text("Press Enter to Start", WIDTH // 2, 430, 40, True)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                return False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_RETURN:
                return True

        pygame.display.update()
        clock.tick(FPS)


def game():
    main_circle = Circle(WIDTH // 2, HEIGHT // 2)

    c1, c2 = None, None
    split = False
    cooldown = 0

    controls_p1 = {
        "up": pygame.K_w,
        "down": pygame.K_s,
        "left": pygame.K_a,
        "right": pygame.K_d
    }

    controls_p2 = {
        "up": pygame.K_UP,
        "down": pygame.K_DOWN,
        "left": pygame.K_LEFT,
        "right": pygame.K_RIGHT
    }

    running = True
    while running:
        win.fill(BG)
        keys = pygame.key.get_pressed()

        if cooldown > 0:
            cooldown -= 1

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and not split:
                    split = True
                    cooldown = 20

                    c1 = Circle(main_circle.x - 25, main_circle.y, SMALL_RADIUS, RED)
                    c2 = Circle(main_circle.x + 25, main_circle.y, SMALL_RADIUS, RED)

        if not split:
            main_circle.move(keys, controls_p1)
            main_circle.draw()
        else:
            c1.move(keys, controls_p1)
            c2.move(keys, controls_p2)

            c1.draw(1)
            c2.draw(2)

            distance = math.hypot(c1.x - c2.x, c1.y - c2.y)
            if distance < (c1.r + c2.r + 6) and cooldown == 0:
                main_circle = Circle(
                    (c1.x + c2.x) / 2,
                    (c1.y + c2.y) / 2
                )
                split = False
                c1, c2 = None, None

        pygame.display.update()
        clock.tick(FPS)


# Run the game
if start_screen():
    game()

pygame.quit()
