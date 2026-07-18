import pygame
import random
import sys
import math

# 1. إعدادات النظام والألوان النيونية
WIDTH = 800
HEIGHT = 600
FPS = 60

# الألوان (RGB)
BLACK = (10, 10, 15)
DARK_GRAY = (40, 40, 45)
WHITE = (255, 255, 255)
NEON_BLUE = (0, 191, 255)      # لون الكرة الأساسي
NEON_PURPLE = (148, 0, 211)    # عند جمع الطاقة
NEON_GREEN = (57, 255, 20)     # الدرع
NEON_RED = (255, 7, 58)        # العقبات
NEON_CYAN = (0, 255, 243)      # البلورات

class ParticleSystem:
    """نظام الجزيئات لإضافة تأثيرات الانفجار وذيل الحركة"""
    def __init__(self):
        self.particles = []

    def emit(self, x, y, color, amount=1, speed=2):
        for _ in range(amount):
            angle = random.uniform(0, math.pi * 2)
            s = random.uniform(1, speed)
            self.particles.append({
                "x": x, "y": y,
                "vx": math.cos(angle) * s,
                "vy": math.sin(angle) * s,
                "radius": random.randint(3, 6),
                "color": color,
                "alpha": 255
            })

    def update_and_draw(self, surface):
        for p in self.particles[:]:
            p["x"] += p["vx"]
            p["y"] += p["vy"]
            p["alpha"] -= 5
            p["radius"] -= 0.05
            if p["alpha"] <= 0 or p["radius"] <= 0:
                self.particles.remove(p)
                continue
            
            # رسم الجزيء مع تأثير الشفافية
            s = pygame.Surface((p["radius"]*2, p["radius"]*2), pygame.SRCALPHA)
            pygame.draw.circle(s, (*p["color"], p["alpha"]), (p["radius"], p["radius"]), p["radius"])
            surface.blit(s, (p["x"] - p["radius"], p["y"] - p["radius"]))

class Player:
    def __init__(self):
        self.x = 150
        self.y = 300
        self.radius = 18
        self.velocity = 0
        self.gravity = 0.6
        self.jump_strength = -10
        self.state = "NORMAL" # NORMAL, ENERGY, SHIELD
        self.color = NEON_BLUE
        self.shield_active = False

    def jump(self):
        self.velocity = self.jump_strength

    def update(self):
        self.velocity += self.gravity
        self.y += self.velocity
        
        # تغيير الألوان حسب الحالة
        if self.state == "SHIELD":
            self.color = NEON_GREEN
        elif self.state == "ENERGY":
            self.color = NEON_PURPLE
        else:
            self.color = NEON_BLUE

        # حدود الشاشة (السقف والأرضية)
        if self.y < self.radius:
            self.y = self.radius
            self.velocity = 0
        if self.y > HEIGHT - 50 - self.radius:
            self.y = HEIGHT - 50 - self.radius
            self.velocity = 0

    def draw(self, surface):
        # تأثير التوهج (رسم دوائر أكبر شفافة خلف الكرة)
        for i in range(3, 0, -1):
            glow_surf = pygame.Surface((self.radius*2 + i*8, self.radius*2 + i*8), pygame.SRCALPHA)
            pygame.draw.circle(glow_surf, (*self.color, 50 // i), (self.radius + i*4, self.radius + i*4), self.radius + i*4)
            surface.blit(glow_surf, (self.x - self.radius - i*4, self.y - self.radius - i*4))
        
        # الكرة الأساسية
        pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), self.radius)

class Obstacle:
    def __init__(self, speed):
        self.x = WIDTH + 50
        self.width = random.randint(40, 70)
        self.height = random.randint(150, 300)
        self.type = random.choice(["TOP", "BOTTOM", "MOVING"])
        self.speed = speed
        self.y = 0 if self.type == "TOP" else HEIGHT - 50 - self.height
        self.move_dir = 1 if self.type == "MOVING" else 0
        
        if self.type == "MOVING":
            self.y = random.randint(100, 300)
            self.height = 100

    def update(self):
        self.x -= self.speed
        if self.type == "MOVING":
            self.y += self.move_dir * 2
            if self.y < 50 or self.y > HEIGHT - 200:
                self.move_dir *= -1

    def draw(self, surface):
        rect = pygame.Rect(self.x, self.y, self.width, self.height)
        # رسم العقبة باللون الأحمر النيوني
        pygame.draw.rect(surface, NEON_RED, rect, border_radius=5)
        # خط خارجي متوهج
        pygame.draw.rect(surface, WHITE, rect, 2, border_radius=5)

class Collectible:
    def __init__(self, speed):
        self.x = WIDTH + 100
        self.y = random.randint(100, HEIGHT - 150)
        self.type = random.choice(["CRYSTAL", "SHIELD"])
        self.speed = speed
        self.radius = 10
        self.color = NEON_CYAN if self.type == "CRYSTAL" else NEON_GREEN

    def update(self):
        self.x -= self.speed

    def draw(self, surface):
        if self.type == "CRYSTAL":
            # رسم شكل ماسي للبلورة
            points = [(self.x, self.y - self.radius), (self.x + self.radius, self.y), 
                      (self.x, self.y + self.radius), (self.x - self.radius, self.y)]
            pygame.draw.polygon(surface, self.color, points)
        else:
            # رسم درع دائري
            pygame.draw.circle(surface, self.color, (int(self.x), int(self.y)), self.radius, 2)
            pygame.draw.circle(surface, WHITE, (int(self.x), int(self.y)), self.radius - 4)

# 2. فئة اللعبة الرئيسية لإدارة الشاشات والمراحل
class Game:
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption("Color Dash: Escape the Void")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont("Arial", 30, bold=True)
        self.large_font = pygame.font.SysFont("Arial", 60, bold=True)
        self.screen_shake = 0
        
        self.state = "MENU" # MENU, PLAYING, GAME_OVER
        self.high_score = 0
        self.stars = [] # خلفية النجوم المارقة
        self.init_stars()

    def init_stars(self):
        self.stars = [{"x": random.randint(0, WIDTH), "y": random.randint(0, HEIGHT), "speed": random.uniform(0.5, 2)} for _ in range(50)]

    def reset_game(self):
        self.player = Player()
        self.particles = ParticleSystem()
        self.obstacles = []
        self.collectibles = []
        self.score = 0
        self.crystals_collected = 0
        self.game_speed = 5
        self.spawn_timer = 0
        self.difficulty_timer = pygame.time.get_ticks()
        self.screen_shake = 0

    def handle_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN or (event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE):
                if self.state == "MENU":
                    self.reset_game()
                    self.state = "PLAYING"
                elif self.state == "PLAYING":
                    self.player.jump()
                    self.particles.emit(self.player.x, self.player.y, self.player.color, amount=3, speed=1)
                elif self.state == "GAME_OVER":
                    self.state = "MENU"

    def update(self):
        # تحريك نجوم الخلفية
        for star in self.stars:
            star["x"] -= star["speed"]
            if star["x"] < 0:
                star["x"] = WIDTH

        if self.state == "PLAYING":
            self.player.update()
            
            # تأثير ذيل الحركة النيونية للكرة
            if random.random() > 0.4:
                self.particles.emit(self.player.x - 10, self.player.y, self.player.color, amount=1, speed=0.5)

            # زيادة الصعوبة كل 20 ثانية
            if pygame.time.get_ticks() - self.difficulty_timer > 20000:
                self.game_speed += 0.8
                self.difficulty_timer = pygame.time.get_ticks()

            # توليد العقبات والبلورات
            self.spawn_timer += 1
            if self.spawn_timer > random.randint(70, 110):
                if random.random() > 0.3:
                    self.obstacles.append(Obstacle(self.game_speed))
                else:
                    self.collectibles.append(Collectible(self.game_speed))
                self.spawn_timer = 0

            # تحديث العقبات والاصطدامات
            for obs in self.obstacles[:]:
                obs.update()
                if obs.x < -100:
                    self.obstacles.remove(obs)
                    self.score += 5 # نقاط لتجاوز العقبة
                
                # فحص اصطدام اللاعب بالعقبة
                player_rect = pygame.Rect(self.player.x - self.player.radius, self.player.y - self.player.radius, self.player.radius*2, self.player.radius*2)
                obs_rect = pygame.Rect(obs.x, obs.y, obs.width, obs.height)
                if player_rect.colliderect(obs_rect):
                    if self.player.shield_active:
                        self.player.shield_active = False
                        self.player.state = "NORMAL"
                        self.obstacles.remove(obs)
                        self.screen_shake = 15 # اهتزاز خفيف
                    else:
                        self.particles.emit(self.player.x, self.player.y, NEON_RED, amount=40, speed=5)
                        self.state = "GAME_OVER"
                        self.screen_shake = 30 # اهتزاز قوي عند الموت
                        if self.score > self.high_score:
                            self.high_score = self.score

            # تحديث العناصر القابلة للجمع
            for col in self.collectibles[:]:
                col.update()
                if col.x < -50:
                    self.collectibles.remove(col)
                
                # المسافة بين اللاعب والعنصر لجمعها بدقة دائرية
                distance = math.hypot(self.player.x - col.x, self.player.y - col.y)
                if distance < self.player.radius + col.radius:
                    if col.type == "CRYSTAL":
                        self.crystals_collected += 1
                        self.score += 10
                        self.particles.emit(col.x, col.y, NEON_CYAN, amount=10, speed=3)
                    elif col.type == "SHIELD":
                        self.player.shield_active = True
                        self.player.state = "SHIELD"
                        self.particles.emit(col.x, col.y, NEON_GREEN, amount=10, speed=3)
                    self.collectibles.remove(col)

            self.particles.update_and_draw(self.screen)

    def draw(self):
        # إعداد سطح رسم مهتز في حال وجود اهتزاز شاشة
        render_surf = pygame.Surface((WIDTH, HEIGHT))
        render_surf.fill(BLACK)

        # رسم النجوم
        for star in self.stars:
            pygame.draw.circle(render_surf, (150, 150, 150), (int(star["x"]), int(star["y"])), 1)

        if self.state == "MENU":
            title = self.large_font.render("COLOR DASH", True, NEON_BLUE)
            subtitle = self.font.render("Escape the Void", True, NEON_PURPLE)
            prompt = self.font.render("Click anywhere to PLAY", True, WHITE)
            render_surf.blit(title, (WIDTH//2 - title.get_width()//2, 150))
            render_surf.blit(subtitle, (WIDTH//2 - subtitle.get_width()//2, 220))
            render_surf.blit(prompt, (WIDTH//2 - prompt.get_width()//2, 400))

        elif self.state == "PLAYING":
            # رسم الأرضية
            pygame.draw.rect(render_surf, DARK_GRAY, (0, HEIGHT - 50, WIDTH, 50))
            pygame.draw.line(render_surf, NEON_BLUE, (0, HEIGHT - 50), (WIDTH, HEIGHT - 50), 2)

            for obs in self.obstacles: obs.draw(render_surf)
            for col in self.collectibles: col.draw(render_surf)
            self.player.draw(render_surf)
            self.particles.update_and_draw(render_surf)

            # واجهة المستخدم العلوية (HUD)
            score_txt = self.font.render(f"Score: {self.score}", True, WHITE)
            crystal_txt = self.font.render(f"💎: {self.crystals_collected}", True, NEON_CYAN)
            render_surf.blit(score_txt, (20, 20))
            render_surf.blit(crystal_txt, (WIDTH - 150, 20))

        elif self.state == "GAME_OVER":
            go_txt = self.large_font.render("GAME OVER", True, NEON_RED)
            sc_txt = self.font.render(f"Your Score: {self.score}", True, WHITE)
            best_txt = self.font.render(f"Best Score: {self.high_score}", True, NEON_GREEN)
            retry_txt = self.font.render("Click to Return to Menu", True, DARK_GRAY)
            
            render_surf.blit(go_txt, (WIDTH//2 - go_txt.get_width()//2, 150))
            render_surf.blit(sc_txt, (WIDTH//2 - sc_txt.get_width()//2, 250))
            render_surf.blit(best_txt, (WIDTH//2 - best_txt.get_width()//2, 310))
            render_surf.blit(retry_txt, (WIDTH//2 - retry_txt.get_width()//2, 450))

        # تطبيق اهتزاز الشاشة (Screen Shake)
        shake_x = 0
        shake_y = 0
        if self.screen_shake > 0:
            shake_x = random.randint(-self.screen_shake, self.screen_shake)
            shake_y = random.randint(-self.screen_shake, self.screen_shake)
            self.screen_shake -= 1

        self.screen.blit(render_surf, (shake_x, shake_y))
        pygame.display.flip()

    def run(self):
        while True:
            self.handle_events()
            self.update()
            self.draw()
            self.clock.tick(FPS)

if __name__ == "__main__":
    game = Game()
    game.run()
