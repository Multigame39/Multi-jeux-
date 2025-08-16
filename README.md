import pygame
import random
import sys
import json
import datetime

pygame.init()

# FULLSCREEN
screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
W, H = screen.get_size()
pygame.display.set_caption("Menu multi-jeux")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)
bigfont = pygame.font.SysFont(None, 48)

# Couleurs
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
PURPLE = (128, 0, 128)
GRAY = (200, 200, 200)
RED = (200, 0, 0)
GREEN = (0, 200, 0)
BLUE = (0, 0, 200)

# Chargement des scores
try:
    with open("scores.json", "r") as f:
        all_scores = json.load(f)
except:
    all_scores = {"devine": 0, "flappy": 0, "subway": 0, "tir": 0}

def save_scores():
    with open("scores.json", "w") as f:
        json.dump(all_scores, f)

def draw_quit_button():
    btn_rect = pygame.Rect(W - 120, 10, 110, 50)
    pygame.draw.rect(screen, PURPLE, btn_rect)
    txt = font.render("Quitter", True, WHITE)
    screen.blit(txt, (W - 110, 20))
    return btn_rect

class Button:
    def __init__(self, rect, color, text):
        self.rect = pygame.Rect(rect)
        self.color = color
        self.text = text
        self.txt_surface = font.render(text, True, WHITE)

    def draw(self, surf):
        pygame.draw.rect(surf, self.color, self.rect)
        txt_rect = self.txt_surface.get_rect(center=self.rect.center)
        surf.blit(self.txt_surface, txt_rect)

    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)

def menu(app_start_ticks):
    buttons = [
        Button((W//2 - 150, H//3, 300, 70), BLUE, "Devine le nombre"),
        Button((W//2 - 150, H//3 + 100, 300, 70), GREEN, "Flappy Bird"),
        Button((W//2 - 150, H//3 + 200, 300, 70), RED, "Subway Surfer"),
        Button((W//2 - 150, H//3 + 300, 300, 70), PURPLE, "Tir sur personnages"),
        Button((W//2 - 150, H//3 + 400, 300, 70), BLACK, "Dessin"),
    ]

    while True:
        screen.fill(WHITE)
        title = bigfont.render("Menu Multi-Jeux", True, BLACK)
        screen.blit(title, (W//2 - title.get_width()//2, H//5))

        # Date et heure
        now = datetime.datetime.now()
        date_str = now.strftime("%H:%M:%S  %d/%m/%Y")
        date_surface = font.render(date_str, True, BLACK)
        screen.blit(date_surface, (10, 10))

        # Temps passé
        elapsed_ms = pygame.time.get_ticks() - app_start_ticks
        elapsed_sec = elapsed_ms // 1000
        minutes = elapsed_sec // 60
        seconds = elapsed_sec % 60
        time_str = f"Temps passé : {minutes}m{seconds}s"
        time_surface = font.render(time_str, True, BLACK)
        screen.blit(time_surface, (10, H - 40))

        quit_btn = draw_quit_button()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                pos = event.pos
                if quit_btn.collidepoint(pos):
                    save_scores()
                    pygame.quit()
                    sys.exit()
                for i, btn in enumerate(buttons):
                    if btn.is_clicked(pos):
                        return i

        for btn in buttons:
            btn.draw(screen)

        pygame.display.flip()
        clock.tick(60)

def devine_le_nombre():
    number_to_guess = random.randint(1, 100)
    guess = ""
    message = "Devine un nombre entre 1 et 100"
    error_msg = ""
    won = False
    attempts = 0

    keys = []
    key_w, key_h = 80, 80
    margin = 15
    start_x = (W - (key_w*3 + margin*2))//2
    start_y = H//2

    for i in range(9):
        x = start_x + (i % 3) * (key_w + margin)
        y = start_y + (i // 3) * (key_h + margin)
        keys.append((str(i+1), pygame.Rect(x, y, key_w, key_h)))
    keys.append(("0", pygame.Rect(start_x + key_w + margin, start_y + 3*(key_h + margin), key_w, key_h)))
    keys.append(("OK", pygame.Rect(start_x + 2*(key_w + margin), start_y + 3*(key_h + margin), key_w, key_h)))
    keys.append(("DEL", pygame.Rect(start_x, start_y + 3*(key_h + margin), key_w, key_h)))

    while True:
        screen.fill(WHITE)
        quit_btn = draw_quit_button()

        msg_surface = font.render(message, True, BLACK)
        screen.blit(msg_surface, (W//2 - msg_surface.get_width()//2, H//6))

        guess_surface = bigfont.render(guess, True, BLACK)
        screen.blit(guess_surface, (W//2 - guess_surface.get_width()//2, H//6 + 60))

        err_surface = font.render(error_msg, True, RED)
        screen.blit(err_surface, (W//2 - err_surface.get_width()//2, H//6 + 110))

        for label, rect in keys:
            pygame.draw.rect(screen, GRAY, rect)
            txt = font.render(label, True, BLACK)
            txt_rect = txt.get_rect(center=rect.center)
            screen.blit(txt, txt_rect)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                pos = event.pos
                if quit_btn.collidepoint(pos):
                    return
                for label, rect in keys:
                    if rect.collidepoint(pos):
                        if label == "DEL":
                            guess = guess[:-1]
                            error_msg = ""
                        elif label == "OK":
                            if guess == "":
                                error_msg = "Entre un nombre !"
                            else:
                                val = int(guess)
                                attempts += 1
                                if val == number_to_guess:
                                    message = f"Bravo ! C'était {val} en {attempts} essais"
                                    won = True
                                    if all_scores["devine"] == 0 or attempts < all_scores["devine"]:
                                        all_scores["devine"] = attempts
                                        save_scores()
                                elif val < number_to_guess:
                                    message = "Trop petit !"
                                else:
                                    message = "Trop grand !"
                                guess = ""
                        else:
                            if len(guess) < 3:
                                guess += label
                        break

        if won:
            pygame.time.delay(2500)
            return

        pygame.display.flip()
        clock.tick(60)

def flappy_bird():
    bird = pygame.Rect(50, H//2, 30, 30)
    gravity = 0.5
    velocity = 0
    jump_power = -10

    pipe_width = 70
    pipe_gap = 150
    pipes = []
    pipe_speed = 4
    spawn_timer = 0
    score = 0
    font_small = pygame.font.SysFont(None, 28)

    def create_pipe():
        y = random.randint(100, H - 100 - pipe_gap)
        top_pipe = pygame.Rect(W, 0, pipe_width, y)
        bottom_pipe = pygame.Rect(W, y + pipe_gap, pipe_width, H - y - pipe_gap)
        return (top_pipe, bottom_pipe)

    pipes.append(create_pipe())

    running = True
    while running:
        screen.fill(WHITE)
        quit_btn = draw_quit_button()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                if quit_btn.collidepoint(event.pos):
                    return
                velocity = jump_power

        velocity += gravity
        bird.y += int(velocity)

        for pipe_pair in pipes:
            pipe_pair[0].x -= pipe_speed
            pipe_pair[1].x -= pipe_speed

        if pipes and pipes[0][0].right < 0:
            pipes.pop(0)
            score += 1

        spawn_timer += 1
        if spawn_timer > 90:
            pipes.append(create_pipe())
            spawn_timer = 0

        if bird.top < 0 or bird.bottom > H:
            running = False

        for pipe_pair in pipes:
            if bird.colliderect(pipe_pair[0]) or bird.colliderect(pipe_pair[1]):
                running = False

        pygame.draw.rect(screen, BLUE, bird)

        for pipe_pair in pipes:
            pygame.draw.rect(screen, GREEN, pipe_pair[0])
            pygame.draw.rect(screen, GREEN, pipe_pair[1])

        score_txt = font_small.render(f"Score : {score}", True, BLACK)
        screen.blit(score_txt, (10, 10))

        pygame.display.flip()
        clock.tick(60)

    # Mettre à jour le meilleur score flappy
    if score > all_scores["flappy"]:
        all_scores["flappy"] = score
        save_scores()
    pygame.time.delay(1500)

def subway_surfer():
    # Jeu simple de déplacement gauche/droite pour éviter obstacles, touches flèches ou clics
    player = pygame.Rect(W//2 - 25, H - 100, 50, 50)
    player_speed = 10
    obstacles = []
    obstacle_width = 50
    obstacle_height = 50
    obstacle_speed = 7
    spawn_timer = 0
    score = 0
    font_small = pygame.font.SysFont(None, 28)

    running = True
    while running:
        screen.fill(WHITE)
        quit_btn = draw_quit_button()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                if quit_btn.collidepoint(event.pos):
                    return
                # On détecte clic gauche ou droite pour déplacement
                if event.pos[0] < W//2:
                    player.x -= player_speed
                else:
                    player.x += player_speed
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT:
                    player.x -= player_speed
                elif event.key == pygame.K_RIGHT:
                    player.x += player_speed

        # Gérer limites écran
        if player.left < 0:
            player.left = 0
        if player.right > W:
            player.right = W

        # Générer obstacles
        spawn_timer += 1
        if spawn_timer > 40:
            x_pos = random.choice([W//4 - obstacle_width//2, W//2 - obstacle_width//2, 3*W//4 - obstacle_width//2])
            obstacles.append(pygame.Rect(x_pos, -obstacle_height, obstacle_width, obstacle_height))
            spawn_timer = 0

        # Déplacer obstacles
        for obs in obstacles:
            obs.y += obstacle_speed

        # Supprimer obstacles hors écran
        obstacles = [o for o in obstacles if o.y < H + obstacle_height]

        # Vérifier collisions
        for obs in obstacles:
            if player.colliderect(obs):
                running = False

        pygame.draw.rect(screen, BLUE, player)
        for obs in obstacles:
            pygame.draw.rect(screen, RED, obs)

        score += 1
        score_txt = font_small.render(f"Score : {score//10}", True, BLACK)
        screen.blit(score_txt, (10, 10))

        pygame.display.flip()
        clock.tick(60)

    if score//10 > all_scores["subway"]:
        all_scores["subway"] = score//10
        save_scores()
    pygame.time.delay(1500)

def tir_sur_personnages():
    # Jeu où il faut cliquer sur des cibles qui apparaissent,
    # si on rate une cible (elle disparaît sans être cliquée), on perd
    target_size = 60
    target = pygame.Rect(random.randint(50, W - 50 - target_size), random.randint(50, H - 50 - target_size), target_size, target_size)
    score = 0
    timer = 0
    max_time = 2000  # ms pour cliquer la cible

    font_small = pygame.font.SysFont(None, 28)

    running = True
    last_spawn = pygame.time.get_ticks()

    while running:
        screen.fill(WHITE)
        quit_btn = draw_quit_button()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                if quit_btn.collidepoint(event.pos):
                    return
                if target.collidepoint(event.pos):
                    score += 1
                    target.x = random.randint(50, W - 50 - target_size)
                    target.y = random.randint(50, H - 50 - target_size)
                    last_spawn = pygame.time.get_ticks()
                else:
                    # Miss click => perdre
                    running = False

        now = pygame.time.get_ticks()
        if now - last_spawn > max_time:
            # Trop tard => perdre
            running = False

        pygame.draw.rect(screen, RED, target)
        score_txt = font_small.render(f"Score : {score}", True, BLACK)
        screen.blit(score_txt, (10, 10))

        pygame.display.flip()
        clock.tick(60)

    if score > all_scores["tir"]:
        all_scores["tir"] = score
        save_scores()
    pygame.time.delay(1500)

def dessin():
    # Jeu de dessin simple avec choix de couleurs
    drawing = False
    brush_size = 8
    current_color = BLACK
    points = []

    # Boutons couleurs
    colors = [RED, BLUE, BLACK]
    btns = []
    btn_size = 50
    margin = 10
    for i, c in enumerate(colors):
        btns.append(pygame.Rect(10 + i*(btn_size + margin), H - btn_size - 10, btn_size, btn_size))

    quit_btn = draw_quit_button()

    running = True
    while running:
        screen.fill(WHITE)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_scores()
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                if quit_btn.collidepoint(event.pos):
                    return
                for i, btn in enumerate(btns):
                    if btn.collidepoint(event.pos):
                        current_color = colors[i]
                drawing = True
                points.append((event.pos, current_color))
            elif event.type == pygame.MOUSEBUTTONUP:
                drawing = False
            elif event.type == pygame.MOUSEMOTION and drawing:
                points.append((event.pos, current_color))

        for pos, col in points:
            pygame.draw.circle(screen, col, pos, brush_size)

        for i, btn in enumerate(btns):
            pygame.draw.rect(screen, colors[i], btn)
            if colors[i] == current_color:
                pygame.draw.rect(screen, PURPLE, btn, 4)
            else:
                pygame.draw.rect(screen, BLACK, btn, 2)

        quit_btn = draw_quit_button()
        pygame.display.flip()
        clock.tick(60)

def main():
    app_start_ticks = pygame.time.get_ticks()

    while True:
        choice = menu(app_start_ticks)
        if choice == 0:
            devine_le_nombre()
        elif choice == 1:
            flappy_bird()
        elif choice == 2:
            subway_surfer()
        elif choice == 3:
            tir_sur_personnages()
        elif choice == 4:
            dessin()

if __name__ == "__main__":
    main()
