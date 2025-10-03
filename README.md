# Chimpanze-Memory-Game
import pygame
import random
import time
import os

pygame.init()

# Screen setup
WIDTH, HEIGHT = 600, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Chimpanzee Memory Game")

FONT = pygame.font.SysFont(None, 48)
clock = pygame.time.Clock()

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GRAY = (150, 150, 150)
BLUE = (0, 150, 255)
YELLOW = (255, 255, 0)
RED = (200, 0, 0)
GREEN = (0, 200, 0)

# Load sound effects
click_sound = pygame.mixer.Sound(r"C:\Users\mohan\Downloads\click-tap-computer-mouse-352734.mp3")
correct_sound = pygame.mixer.Sound(r"C:\Users\mohan\Downloads\bubble-pop-06-351337.mp3")
wrong_sound = pygame.mixer.Sound(r"C:\Users\mohan\Downloads\transition-explosion-121425.mp3")

def generate_positions(n):
    positions = []
    size = 80
    while len(positions) < n:
        x = random.randint(50, WIDTH - 130)
        y = random.randint(50, HEIGHT - 130)
        rect = pygame.Rect(x, y, size, size)
        if not any(r.colliderect(rect.inflate(20, 20)) for r in positions):
            positions.append(rect)
    return positions

def draw_boxes(numbers, hide=False, highlight=None, clicked=[]):
    for i, rect in enumerate(boxes):
        if i in clicked:
            color = (100, 100, 100)  # Fainted gray for already clicked
        elif highlight == i:
            color = YELLOW
        elif hide:
            color = GRAY
        else:
            color = BLUE

        pygame.draw.rect(screen, color, rect, border_radius=8)

        if not hide and i not in clicked:
            num = FONT.render(str(numbers[i]), True, WHITE)
            screen.blit(num, (rect.x + 25, rect.y + 20))

def show_message(msg, color=WHITE):
    screen.fill(BLACK)
    text = FONT.render(msg, True, color)
    screen.blit(text, (WIDTH // 2 - text.get_width() // 2, HEIGHT // 2))
    pygame.display.update()
    time.sleep(2)

# Game loop
level = 3
running = True

while running:
    screen.fill(BLACK)
    numbers = list(range(1, level + 1))
    random.shuffle(numbers)
    boxes = generate_positions(level)

    draw_boxes(numbers)
    pygame.display.update()
    pygame.time.delay(1000)

    draw_boxes(numbers, hide=True)
    pygame.display.update()

    clicked = []
    waiting = True

    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                waiting = False

            if event.type == pygame.MOUSEBUTTONDOWN:
                for i, rect in enumerate(boxes):
                    if rect.collidepoint(event.pos) and i not in clicked:
                        click_sound.play()
                        clicked.append(i)

                        # Redraw with highlight
                        screen.fill(BLACK)
                        draw_boxes(numbers, hide=True, highlight=i, clicked=clicked)
                        pygame.display.update()

                        if numbers[i] != len(clicked):
                            wrong_sound.play()
                            show_message("Wrong! Game Over.", RED)
                            level = 3
                            waiting = False
                            break
                        elif len(clicked) == level:
                            correct_sound.play()
                            show_message("Correct!", GREEN)
                            level += 1
                            waiting = False
                            break

        clock.tick(30)

pygame.quit()
