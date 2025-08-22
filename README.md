import pygame, sys, time, random

pygame.init()

# ألوان
WHITE = (255, 255, 255)
YELLOW = (255, 255, 0)
GREEN = (0, 200, 0)
RED = (200, 0, 0)

# حجم الشاشة
SCREEN_W, SCREEN_H = 800, 600
screen = pygame.display.set_mode((SCREEN_W, SCREEN_H))
pygame.display.set_caption("Alaoui Mobile")

FONT = pygame.font.SysFont(None, 36)

# خلفية البداية
try:
    bg = pygame.image.load("assets/start_bg.png")
    bg = pygame.transform.scale(bg, (SCREEN_W, SCREEN_H))
except:
    bg = None

# خلفية اللعبة
try:
    game_bg = pygame.image.load("assets/game_bg.png")
    game_bg = pygame.transform.scale(game_bg, (SCREEN_W, SCREEN_H))
except:
    game_bg = None

# موسيقى الخلفية
try:
    pygame.mixer.music.load("assets/menu_music.mp3")
    pygame.mixer.music.play(-1)
except:
    pass

# أصوات
try:
    shoot_sound = pygame.mixer.Sound("assets/shoot.wav")
    hit_sound = pygame.mixer.Sound("assets/hit.wav")
except:
    shoot_sound = hit_sound = None

# شاشة البداية
def start_screen():
    title_font = pygame.font.SysFont(None, 70)
    sub_font = pygame.font.SysFont(None, 36)

    title = title_font.render("Alaoui Mobile", True, YELLOW)
    sub = sub_font.render("Battle Royale 2D", True, WHITE)
    instructions = FONT.render("اضغط أي زر للبدء", True, GREEN)
    quit_text = FONT.render("اضغط Q للخروج", True, RED)

    show = True
    last_toggle = time.time()
    waiting = True

    while waiting:
        if bg:
            screen.blit(bg, (0,0))
        else:
            screen.fill((15, 20, 30))

        screen.blit(title, (SCREEN_W//2 - title.get_width()//2, SCREEN_H//2 - 120))
        screen.blit(sub, (SCREEN_W//2 - sub.get_width()//2, SCREEN_H//2 - 70))

        if time.time() - last_toggle > 0.5:
            show = not show
            last_toggle = time.time()
        if show:
            screen.blit(instructions, (SCREEN_W//2 - instructions.get_width()//2, SCREEN_H//2 + 60))

        screen.blit(quit_text, (SCREEN_W//2 - quit_text.get_width()//2, SCREEN_H//2 + 110))

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()
                else:
                    pygame.mixer.music.stop()
                    waiting = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                pygame.mixer.music.stop()
                waiting = False

# اللعبة
def main_game():
    player = pygame.Rect(400, 300, 50, 50)
    player_health = 3
    score = 0

    # أعداء متعددين
    enemies = [pygame.Rect(random.randint(0, 750), random.randint(0, 550), 50, 50) for _ in range(5)]
    enemy_health = [2 for _ in range(5)]

    bullets = []

    clock = pygame.time.Clock()
    running = True
    base_enemy_speed = 2
    font = pygame.font.SysFont(None, 36)

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                bullets.append(pygame.Rect(player.x + 20, player.y, 10, 5))
                if shoot_sound: shoot_sound.play()

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]: player.x -= 5
        if keys[pygame.K_RIGHT]: player.x += 5
        if keys[pygame.K_UP]: player.y -= 5
        if keys[pygame.K_DOWN]: player.y += 5

        # سرعة الأعداء تزيد مع الوقت
        enemy_speed = base_enemy_speed + (pygame.time.get_ticks() // 30000)

        # حركة الأعداء
        for i, enemy in enumerate(enemies):
            if enemy.x < player.x: enemy.x += enemy_speed
            if enemy.x > player.x: enemy.x -= enemy_speed
            if enemy.y < player.y: enemy.y += enemy_speed
            if enemy.y > player.y: enemy.y -= enemy_speed

            if enemy.colliderect(player):
                player_health -= 1
                enemy.x, enemy.y = random.randint(0, 750), random.randint(0, 550)
                if player_health <= 0:
                    running = False

        # حركة الرصاصات
        for bullet in bullets[:]:
            bullet.y -= 10
            for i, enemy in enumerate(enemies):
                if bullet.colliderect(enemy):
                    enemy_health[i] -= 1
                    if hit_sound: hit_sound.play()
                    bullets.remove(bullet)
                    if enemy_health[i] <= 0:
                        enemies[i].x, enemies[i].y = random.randint(0, 750), random.randint(0, 550)
                        enemy_health[i] = 2
                        score += 10
                    break
            if bullet.y < 0 and bullet in bullets:
                bullets.remove(bullet)

        # رسم الخلفية
        if game_bg:
            screen.blit(game_bg, (0,0))
        else:
            screen.fill((30, 30, 30))

        # رسم اللاعب والأعداء والرصاصات
        pygame.draw.rect(screen, (0, 255, 0), player)
        for enemy in enemies:
            pygame.draw.rect(screen, (255, 0, 0), enemy)
        for bullet in bullets:
            pygame.draw.rect(screen, (255, 255, 0), bullet)

        # نصوص الصحة والنقاط
        health_text = font.render(f'Health: {player_health}', True, WHITE)
        score_text = font.render(f'Score: {score}', True, WHITE)
        screen.blit(health_text, (10, 10))
        screen.blit(score_text, (10, 40))

        pygame.display.flip()
        clock.tick(60)

    # شاشة النهاية
    game_over_screen(font, score)

def game_over_screen(font, score):
    screen.fill((40, 0, 0))
    over_text = font.render("انتهت اللعبة!", True, YELLOW)
    score_text = font.render(f"نتيجتك: {score}", True, WHITE)
    restart_text = font.render("اضغط R للبدء من جديد أو Q للخروج", True, GREEN)

    screen.blit(over_text, (SCREEN_W//2 - over_text.get_width()//2, SCREEN_H//2 - 80))
    screen.blit(score_text, (SCREEN_W//2 - score_text.get_width()//2, SCREEN_H//2 - 40))
    screen.blit(restart_text, (SCREEN_W//2 - restart_text.get_width()//2, SCREEN_H//2 + 20))
    pygame.display.flip()

    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()
                if event.key == pygame.K_r:
                    waiting = False
                    main_game()

# === تشغيل اللعبة ===
start_screen()
main_game()

def start_screen():
    screen.fill((15, 20, 30))
    title_font = pygame.font.SysFont(None, 70)
    title = title_font.render("Alaoui Mobile", True, YELLOW)
    screen.blit(title, (SCREEN_W//2 - title.get_width()//2, SCREEN_H//2 - 120))
    sub_font = pygame.font.SysFont(None, 36)
    sub = sub_font.render("Battle Royale 2D", True, WHITE)
    screen.blit(sub, (SCREEN_W//2 - sub.get_width()//2, SCREEN_H//2 - 70))
    instructions = FONT.render("اضغط أي زر للبدء", True, GREEN)
    screen.blit(instructions, (SCREEN_W//2 - instructions.get_width()//2, SCREEN_H//2 + 60))
    pygame.display.flip()
    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN or event.type == pygame.MOUSEBUTTONDOWN:
                waiting = False
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

start_screen()


play Alaoui mobile 





....."Creating an Application".......

// mymath.c
#include <math.h>

double fast_hypot(double a, double b) {
    return hypot(a, b);
}
gcc -shared -fPIC -O3 -o libmymath.so mymath.c -lm
# أو على macOS
# clang -shared -fPIC -O3 -o libmymath.dylib mymath.c -lm
cl /LD /O2 mymath.c /Fe:mymath.dll
import ctypes, os, sys
from pathlib import Path

libname = {"win32": "mymath.dll", "darwin": "libmymath.dylib"}.get(sys.platform, "libmymath.so")
libpath = Path(__file__).with_name(libname)
lib = ctypes.CDLL(str(libpath))

lib.fast_hypot.argtypes = (ctypes.c_double, ctypes.c_double)
lib.fast_hypot.restype = ctypes.c_double

print("hypot(3,4) =", lib.fast_hypot(3.0, 4.0))
#include <pybind11/pybind11.h>
#include <pybind11/stl.h>
#include <vector>
namespace py = pybind11;

std::vector<double> scale(const std::vector<double>& v, double k) {
    std::vector<double> out(v.size());
    for (size_t i = 0; i < v.size(); ++i) out[i] = v[i] * k;
    return out;
}

PYBIND11_MODULE(vector_ops, m) {
    m.doc() = "Vector ops bound with pybind11";
    m.def("scale", &scale, "Scale a vector", py::arg("v"), py::arg("k"));
}
[build-system]
requires = ["scikit-build-core>=0.9", "pybind11>=2.11"]
build-backend = "scikit_build_core.build"

[project]
name = "vector-ops"
version = "0.1.0"
requires-python = ">=3.9"
dependencies = []

[tool.scikit-build]
wheel.expand-macos-universal2 = true
cmake_minimum_required(VERSION 3.18)
project(vector_ops LANGUAGES CXX)
find_package(pybind11 REQUIRED)
pybind11_add_module(vector_ops vector_ops.cpp)
set_target_properties(vector_ops PROPERTIES CXX_STANDARD 17 CXX_STANDARD_REQUIRED YES)
python -m pip install .          # يبني الامتداد ويثبته
# أو:
python -m build                   # يبني wheel/ sdist داخل dist/
python -m pip install dist/*.whl  # تثبيت الحزمة
import vector_ops
print(vector_ops.scale([1,2,3], 2.5))  # [2.5, 5.0, 7.5]
from PySide6.QtWidgets import QApplication, QWidget, QVBoxLayout, QPushButton, QLabel
import vector_ops, sys

app = QApplication(sys.argv)
w = QWidget(); w.setWindowTitle("Hybrid App")
layout = QVBoxLayout(w)
label = QLabel("Click to scale [1,2,3] by 3.0")
btn = QPushButton("Run"); layout.addWidget(label); layout.addWidget(btn)

def on_click():
    result = vector_ops.scale([1,2,3], 3.0)
    label.setText(str(result))

btn.clicked.connect(on_click)
w.show()
sys.exit(app.exec())
pyinstaller --onefile app.py
