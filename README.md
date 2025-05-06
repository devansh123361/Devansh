import pygame
import random
import sys

# Initialize pygame
pygame.init()

# Game constants
WIDTH, HEIGHT = 400, 600
FPS = 60
GRAVITY = 0.25
BIRD_JUMP = -5
PIPE_SPEED = 3
PIPE_GAP = 150
PIPE_FREQUENCY = 1500  # milliseconds

# Colors
WHITE = (255, 255, 255)
BLUE = (0, 0, 255)
GREEN = (0, 255, 0)
BLACK = (0, 0, 0)
SKY_BLUE = (135, 206, 235)

# Create the game window
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pluffy Bird")
clock = pygame.time.Clock()

# Game variables
score = 0
font = pygame.font.SysFont('Arial', 30)

class Bird:
    def __init__(self):
        self.x = 100
        self.y = HEIGHT // 2
        self.velocity = 0
        self.size = 20
        
    def jump(self):
        self.velocity = BIRD_JUMP
        
    def update(self):
        # Apply gravity
        self.velocity += GRAVITY
        self.y += self.velocity
        
        # Keep bird on screen
        if self.y < 0:
            self.y = 0
            self.velocity = 0
        if self.y > HEIGHT - self.size:
            self.y = HEIGHT - self.size
            self.velocity = 0
            
    def draw(self):
        pygame.draw.circle(screen, (255, 255, 0), (self.x, int(self.y)), self.size)
        # Eye
        pygame.draw.circle(screen, BLACK, (self.x + 8, int(self.y) - 5), 5)
        pygame.draw.circle(screen, WHITE, (self.x + 8, int(self.y) - 5), 2)
        # Beak
        pygame.draw.polygon(screen, (255, 165, 0), 
                           [(self.x + 20, int(self.y)), 
                            (self.x + 30, int(self.y) - 5), 
                            (self.x + 30, int(self.y) + 5)])
        
    def get_mask(self):
        return pygame.Rect(self.x - self.size, self.y - self.size, 
                          self.size * 2, self.size * 2)

class Pipe:
    def __init__(self):
        self.x = WIDTH
        self.height = random.randint(100, HEIGHT - 100 - PIPE_GAP)
        self.top_pipe = pygame.Rect(self.x, 0, 50, self.height)
        self.bottom_pipe = pygame.Rect(self.x, self.height + PIPE_GAP, 50, HEIGHT - self.height - PIPE_GAP)
        self.passed = False
        
    def update(self):
        self.x -= PIPE_SPEED
        self.top_pipe.x = self.x
        self.bottom_pipe.x = self.x
        
    def draw(self):
        pygame.draw.rect(screen, GREEN, self.top_pipe)
        pygame.draw.rect(screen, GREEN, self.bottom_pipe)
        
    def collide(self, bird):
        bird_rect = bird.get_mask()
        return bird_rect.colliderect(self.top_pipe) or bird_rect.colliderect(self.bottom_pipe)

def draw_score():
    score_text = font.render(f"Score: {score}", True, BLACK)
    screen.blit(score_text, (10, 10))

def game_over_screen():
    screen.fill(SKY_BLUE)
    game_over_text = font.render("Game Over!", True, BLACK)
    score_text = font.render(f"Final Score: {score}", True, BLACK)
    restart_text = font.render("Press R to Restart", True, BLACK)
    
    screen.blit(game_over_text, (WIDTH//2 - game_over_text.get_width()//2, HEIGHT//2 - 50))
    screen.blit(score_text, (WIDTH//2 - score_text.get_width()//2, HEIGHT//2))
    screen.blit(restart_text, (WIDTH//2 - restart_text.get_width()//2, HEIGHT//2 + 50))
    
    pygame.display.update()
    
    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:
                    return True
                if event.key == pygame.K_q:
                    pygame.quit()
                    sys.exit()
        clock.tick(FPS)

def main():
    global score
    
    bird = Bird()
    pipes = []
    last_pipe = pygame.time.get_ticks()
    running = True
    game_active = True
    
    while running:
        clock.tick(FPS)
        
        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE and game_active:
                    bird.jump()
                if event.key == pygame.K_q:
                    running = False
        
        # Background
        screen.fill(SKY_BLUE)
        
        if game_active:
            # Bird update
            bird.update()
            bird.draw()
            
            # Generate pipes
            time_now = pygame.time.get_ticks()
            if time_now - last_pipe > PIPE_FREQUENCY:
                pipes.append(Pipe())
                last_pipe = time_now
                
            # Pipe update and collision
            for pipe in pipes[:]:
                pipe.update()
                pipe.draw()
                
                # Check for collision
                if pipe.collide(bird):
                    game_active = False
                
                # Check if pipe passed
                if pipe.x + 50 < bird.x and not pipe.passed:
                    pipe.passed = True
                    score += 1
                
                # Remove off-screen pipes
                if pipe.x < -50:
                    pipes.remove(pipe)
            
            # Check if bird hit the ground or ceiling
            if bird.y <= 0 or bird.y >= HEIGHT - bird.size:
                game_active = False
        else:
            if game_over_screen():
                # Reset game
                bird = Bird()
                pipes = []
                last_pipe = pygame.time.get_ticks()
                score = 0
                game_active = True
        
        draw_score()
        pygame.display.update()
    
    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
