import pygame
import sys
import random
import time
import threading

background = pygame.transform.scale (pygame.image.load('res/Background.png'), (1080,720)) 
apple = pygame.transform.scale (pygame.image.load('res/Apple.png'), (16,16)) 
logo_music = pygame.transform.scale(pygame.image.load('res/Logo_music.png'), (60, 60)) 

class Game():
    def __init__(self):
        self.screen_width = 1080 
        self.screen_height = 720 
        self.white = pygame.Color(255, 255, 255)  
        self.green = pygame.Color(0, 255, 0) 
        self.red = pygame.Color(255, 0, 0)  
        self.fps = pygame.time.Clock() 
        self.score = 0 
        self.dif = 20
        self.flag = True 
    def control_check(self, change_to):
        for event in pygame.event.get(): 
            if event.type == pygame.KEYDOWN: 
                if event.key == ord('d'): 
                    change_to = "RIGHT"  
                elif event.key == ord('a'):  
                    change_to = "LEFT"  
                elif event.key == ord('w'):  
                    change_to = "UP"  
                elif event.key == ord('s'):  
                    change_to = "DOWN" 
                elif event.key == pygame.K_ESCAPE:  
                    pygame.quit() 
                    exit() 
        return change_to  
    def Screen(self):
        self.main_screen = pygame.display.set_mode((self.screen_width, self.screen_height)) 
        pygame.display.set_caption('В поисках еды')
        pygame.display.set_icon(pygame.image.load('res/Icon.bmp')) 
    def game_over(self):
        pygame.mixer.music.pause() 
        go_font = pygame.font.SysFont('monaco', 100)  
        go_surf = go_font.render('Game over', True, self.red)  
        go_rect = go_surf.get_rect()  
        go_rect.midtop = (540, 300)  
        self.main_screen.blit(go_surf, go_rect)  
        self.Render_Score(0)  
        pygame.display.flip()  
        time.sleep(3)  
        pygame.quit()  
        exit()  
    def refresh_screen(self):
        pygame.display.flip()
        game.fps.tick(game.dif)
        if self.score % 10 == 0 and self.flag == True: 
            game.dif += 4
            self.flag = False 
        else:
            if self.score % 10 != 0: 
                self.flag = True 
    def Render_Score(self, choice=1):
        s_font = pygame.font.SysFont('monaco', 74) 
        s_surf = s_font.render('Счет: {0}'.format(self.score), True, self.white) 
        s_rect = s_surf.get_rect() 
        name = s_font.render("Mystery Skulls - Money", True, self.white) 
        if choice == 1: 
            s_rect.midtop = (120, 20) 
        else:
            s_rect.midtop = (540, 380) 
        self.main_screen.blit(s_surf, s_rect) 
        self.main_screen.blit(name, (460, 20)) 
        self.main_screen.blit(logo_music, (370, 15)) 

class Snake():
    def __init__(self, snake_color):
        self.snake_head_pos = [100, 450]  
        self.snake_body = [[100, 450], [90, 450], [80, 450]]
        self.snake_color = snake_color 
        self.direction = "RIGHT" 
        self.change_to = self.direction 
    def change_head_position(self):
        if self.direction == "RIGHT": 
            self.snake_head_pos[0] += 10 
        elif self.direction == "LEFT": 
            self.snake_head_pos[0] -= 10 
        elif self.direction == "UP": 
            self.snake_head_pos[1] -= 10 
        elif self.direction == "DOWN": 
            self.snake_head_pos[1] += 10 
    def validate_direction_and_change(self):
        if any((self.change_to == "RIGHT" and not self.direction == "LEFT", 
                self.change_to == "LEFT" and not self.direction == "RIGHT", 
                self.change_to == "UP" and not self.direction == "DOWN", 
                self.change_to == "DOWN" and not self.direction == "UP")): 
            self.direction = self.change_to 
    def snake_body_mechanism(self, score, food_pos, screen_width, screen_height):
        self.snake_body.insert(0, list(self.snake_head_pos)) 
        if (self.snake_head_pos[0] == food_pos[0] and self.snake_head_pos[1] == food_pos[1]): 
            food_pos = [random.randrange(1, screen_width/10)*10, random.randrange(1, screen_height/10)*10] #генерируем еду
            while food_pos[1] < 140: 
                food_pos = [random.randrange(1, screen_width / 10) * 10, random.randrange(1, screen_height / 10) * 10] 
            score += 1 
            nyam.play() 
        else:
            self.snake_body.pop() 
        return score, food_pos 
    def check_for_boundaries(self, game_over, screen_width, screen_height):
        if any((
            self.snake_head_pos[0] > screen_width-10 
            or self.snake_head_pos[0] < 0, 
            self.snake_head_pos[1] > screen_height-10 
            or self.snake_head_pos[1] < 130 
                )):
            game_over() 
        for block in self.snake_body[1:]: 
            if (block[0] == self.snake_head_pos[0] and block[1] == self.snake_head_pos[1]): 
                game_over()
    def draw_snake(self, play_surface, surface_color):
        play_surface.blit(background, (0, 0))
        for pos in self.snake_body: 
            pygame.draw.rect(play_surface, self.snake_color, pygame.Rect(pos[0], pos[1], 10, 10)) 

class Food():
    def __init__(self, screen_width, screen_height):
        self.food_size_x = 10 
        self.food_size_y = 10 
        self.food_pos = [random.randrange(1, screen_width/10)*10, random.randrange(1, screen_height/10)*10]
        #print(self.food_pos)
        while self.food_pos[1] < 140: 
            self.food_pos = [random.randrange(1, screen_width / 10) * 10, random.randrange(1, screen_height / 10) * 10]
    def draw_food(self, play_surface):
        play_surface.blit(apple,(self.food_pos[0], self.food_pos[1])) 


game = Game() 
snake = Snake(game.green) 
food = Food(game.screen_width, game.screen_height) 

pygame.init() 
game.Screen() 
nyam = pygame.mixer.Sound('res/apple.ogg') 
pygame.mixer.music.load('res/Sound1.mp3') 
pygame.mixer.music.set_volume(0.4) 
nyam.set_volume(0.08) 
pygame.mixer.music.play(10) 

while True:
    snake.change_to = game.control_check(snake.change_to) 

    snake.validate_direction_and_change() 
    snake.change_head_position() 
    game.score, food.food_pos = snake.snake_body_mechanism(
        game.score, food.food_pos, game.screen_width, game.screen_height) 

    snake.draw_snake(game.main_screen, game.white) 
    food.draw_food(game.main_screen) 

    snake.check_for_boundaries(game.game_over, game.screen_width, game.screen_height) 

    game.Render_Score() 
    game.refresh_screen() 
