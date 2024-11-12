from quopri import ESCAPE

import pygame, sys
from pygame.locals import *
import random
from game_over import gameover_screen

#res for screen
res = (800,800)
screen =pygame.display.set_mode(res)
#initialize
pygame.init()
#clock
clock = pygame.time.Clock()
FPS = 60
#define game variables
tiles =(800/800)
scroll = 0
#backgrounds
bsbg = pygame.image.load("galbg.jpg").convert()
bg = pygame.transform.scale(bsbg,(800,800))
bsbg2 = pygame.image.load("gal2.jpg").convert()
bg2 = pygame.transform.scale(bsbg2,(800,800))
bg33 = pygame.image.load("skyez.jpg").convert()
bg3 = pygame.transform.scale(bg33,(800,800))

position = 0
def background():
    global position
    screen.fill(color=(0, 0, 0))
    screen.blit(bg, (position, 0))
    screen.blit(bg,(bg.get_width()+position,0))
    position -=1
    if abs(position)> bg.get_width():
        position = 0
def background2():
    global position
    screen.fill(color=(0, 0, 0))
    screen.blit(bg2, (position, 0))
    screen.blit(bg2, (bg2.get_width() + position, 0))
    position -= 0.9
    if abs(position) > bg2.get_width():
        position = 0
def background3():
    global position
    screen.fill(color=(0, 0, 0))
    screen.blit(bg3, (position, 0))
    screen.blit(bg3, (bg3.get_width() + position, 0))
    position -= 0.9
    if abs(position) > bg2.get_width():
        position = 0
#creating a recst to use for mouse click collisions i think
rect1 = pygame.Rect((80,200,100,100))
rect2 = pygame.Rect((320,200,100,100))
rect3 = pygame.Rect((200,200,100,100))
def start_slayer():
    while True:

        background2()
        global lastblast
        global warpspeed
        ship = pygame.Rect((380, 380, 40, 40))
        shipsurf = pygame.Surface((ship.w, ship.h))
        shipsurf.fill((0, 0, 0))
        slayershi = pygame.image.load("ship1.png").convert_alpha()  # make the background transparent
        slayership = pygame.transform.scale(slayershi, (40, 40))
        meteimg1 = pygame.image.load("slayersun.png").convert_alpha()
        warperimg = pygame.image.load("warper1.png").convert_alpha()
        # sounds
        blaster = pygame.mixer.Sound("blaster-2-81267.mp3")
        collision = pygame.mixer.Sound("small-rock-break-194553.mp3")
        destroyed = pygame.mixer.Sound("rock-destroy-6409.mp3")
        speedboost = pygame.mixer.Sound("boost-100537.mp3")

        meteorspawnint = 1000  # 5 sec spawn interval
        meteors = []
        # time stuff
        lastspawntime = pygame.time.get_ticks()
        last_counter_update = pygame.time.get_ticks()
        lastwarp = pygame.time.get_ticks()
        survivaltime = 0
        spawnrate = 9000  # seconds
        minspwnint = 1000
        highscore = 0
        # upgrades
        boosters = 0
        warps = []
        pulses = []
        pulsecooldown = 5000
        warpspeed = 2.5
        gameover = False

        font = pygame.font.Font("KnightWarrior-w16n8.otf", 32)

        def areapulse():
            pulse_rect = pygame.Rect(ship.centerx,ship.centery,80,80)
            pulse.append(blast_rect)
            global lastpulse
            timern = pygame.time.get_ticks()
            if timern - lastpulse >= pulsecooldown:
                pulse = pygame.Rect(ship.x + ship.width // 2, ship.y, 5, 10)  # Small laser shape
                pulses.append(pulse)
                lastpulse = timern




        def warp_speed():
            size = random.randint(60, 60)
            x = random.randint(0, 800 - size)
            y = random.randint(0, 800 - size)
            warper_rect = pygame.Rect(x, y, size, size)
            warps.append(warper_rect)

        def warp_activated():
            global warpspeed
            for warper in warps:
                if ship.colliderect(warper):
                    warps.remove(warper)
                    warpspeed += 1
                    speedboost.play()

        def meteorspawn():
            side = random.randint(1, 4)
            size = random.randint(20, 60)
            speed_increment = survivaltime // 10000
            base_speed = 2 + speed_increment
            match (side):
                case 1:  # top
                    y = 0
                    x = random.randint(0, 800 - size)
                    dy = random.randint(base_speed, base_speed + 5)
                    dx = 0
                    meteor = pygame.Rect(x, y, size, size)

                case 2:  # left
                    x = 0
                    y = random.randint(0, 800 - size)
                    dx = random.randint(base_speed, base_speed + 5)
                    dy = 0
                case 3:  # right
                    x = 800 - size
                    y = random.randint(0, 800 - size)
                    dx = -random.randint(base_speed, base_speed + 5)
                    dy = 0

                case _:  # bottom
                    y = 800 - size
                    x = random.randint(0, 800 - size)
                    dy = -random.randint(base_speed, base_speed + 5)
                    dx = 0

            meteor = pygame.Rect(x, y, size, size)
            meteors.append((meteor, dx, dy))

        lasers = []
        laser_speed = -10  # Adjust speed as needed
        meteors_destroyed = []
        meteorsdestroyed = 0
        # cooldown
        lasercooldown = 1000
        lastblast = 0
        lastpulse = 0
        meteors2upgrade = 10
        holdfire_unlocked = False

        def fire_laser():
            laser_rect = pygame.Rect(ship.centerx, ship.top, 5, 15)  # Laser size
            lasers.append(laser_rect)
            global lastblast, holdfire_unlocked
            current_time = pygame.time.get_ticks()

            # Check if enough time has passed since the last shot
            if current_time - lastblast >= lasercooldown:
                laser = pygame.Rect(ship.x + ship.width // 2, ship.y, 5, 10)  # Small laser shape
                lasers.append(laser)
                lastblast = current_time

        clock = pygame.time.Clock()
        while True:

            # movement
            keys = pygame.key.get_pressed()
            for event in pygame.event.get():
                if event.type == QUIT:
                    sys.exit(0)
                elif keys[K_ESCAPE]:
                    sys.exit(0)
            if keys[K_m]:  # Go to main menu
                return False
            if keys[K_w]:
                if ship.y > 0:
                    ship = ship.move(0, -warpspeed)
            if keys[K_s]:
                if ship.y < 760:
                    ship = ship.move(0, warpspeed)
            if keys[K_d]:
                if ship.x < 760:
                    ship = ship.move(warpspeed, 0)
            if keys[K_a]:
                if ship.x > 0:
                    ship = ship.move(-warpspeed, 0)
                # get current time or time passed
            if keys[K_SPACE]:  # Spacebar to fire
                if holdfire_unlocked:
                    # Continuous shooting if upgrade is unlocked
                    fire_laser()
                    blaster.play()
                elif not holdfire_unlocked and pygame.time.get_ticks() - lastblast >= lasercooldown:
                    # Single-shot firing if no upgrade
                    fire_laser()
                    blaster.play()

            timern = pygame.time.get_ticks()
            if not gameover:
                if timern - lastspawntime > meteorspawnint:
                    meteorspawn()
                    lastspawntime = timern

                    # Update survival time counter every second
                if timern - last_counter_update >= 1000:  # Every 1000 milliseconds (1 second)
                    survivaltime += 1000
                    last_counter_update = timern

                if survivaltime % spawnrate == 0 and survivaltime > 0:
                    meteorspawnint = max(minspwnint, meteorspawnint - 500)
                if timern - lastwarp > 10000:
                    warp_speed()
                    lastwarp = timern

                warp_activated()

            for i in range(len(meteors)):
                meteor_rect, dx, dy = meteors[i]
                meteor_rect.x += dx  # Move the meteor
                meteor_rect.y += dy
                # Update laser positions and check for collisions
            for laser in lasers[:]:  # Copy list to safely remove items
                laser.y += laser_speed
                # Remove laser if it goes off-screen
                if laser.bottom < 0:
                    lasers.remove(laser)

                # Check for collision with meteors
                for meteor in meteors[:]:  # Copy list to safely remove items
                    if laser.colliderect(meteor[0]):
                        meteors.remove(meteor)
                        lasers.remove(laser)
                        destroyed.play()
                        meteorsdestroyed += 1  # Increment meteors destroyed
                        break
                # Check if laser upgrade should be unlocked
                if meteorsdestroyed >= meteors2upgrade:
                    holdfire_unlocked = True  # Allow holding space to shoot
                    laser_speed = -15  # Increase laser speed (optional upgrade)

            background2()

            screen.blit(slayership, ship)
            for meteor_rect, dx, dy in meteors:
                # Scale the meteor image according to the current meteor size
                scaled_meteor_image = pygame.transform.scale(meteimg1, (meteor_rect.width, meteor_rect.height))
                # Blit the scaled meteor image at the meteor's position
                screen.blit(scaled_meteor_image, meteor_rect)
            for meteor in meteors:
                if ship.colliderect(meteor[0]):
                    collision.play()
                    gameover = True
            for laser in lasers:
                pygame.draw.rect(screen, (0, 255, 0), laser)
            for warper_rect in warps:
                scaled_warp = pygame.transform.scale(warperimg, (warper_rect.width, warper_rect.height))
                screen.blit(scaled_warp, warper_rect)
            if gameover:
                result = "lose"

                if not gameover_screen(result):
                        return
            if not gameover:
                timer_text = font.render("Survival Time: " + str(survivaltime // 1000) + "s", True, (255, 255, 255))
                screen.blit(timer_text, (10, 10))
                Score = font.render(f"Score: {meteorsdestroyed} ", True, (255, 0, 0))
                screen.blit(Score, (10, 50))
                menu = font.render("Press M to return to the menu", True,(255,255,0))
                screen.blit(menu,(10,85))
            else:
                # Display final survival time
                final_time_text = font.render("Game Over! Final Time: " + str(survivaltime // 1000) + "s", True,
                                              (255, 0, 0))
                screen.blit(final_time_text, (200, 400))

                if survivaltime > highscore:
                    highscore = survivaltime

                # Display longest survival time
                highscore_text = font.render("Longest Survival Time: " + str(highscore // 1000) + "s", True,
                                             (255, 255, 0))
                screen.blit(highscore_text, (200, 450))

                pygame.display.flip()
                pygame.time.wait(3000)
                gameover = False
                ship.topleft = (380, 380)
                meteors.clear()
                lastspawntime = pygame.time.get_ticks()
                survivaltime = 0
                meteorspawnint = 1000
                meteorsdestroyed = 0  # reset upgrade
                holdfire_unlocked = False  # reset holdfire upgrade
                warpspeed = 1
                warps.clear()
                lastwarp = pygame.time.get_ticks()

            pygame.display.flip()
            clock.tick(60)
def easy():
    while True:
        background2()
        global lastblast
        global warpspeed
        ship = pygame.Rect((380, 380, 40, 40))
        shipsurf = pygame.Surface((ship.w, ship.h))
        shipsurf.fill((0, 0, 0))
        slayershi = pygame.image.load("shipforeasy.png").convert_alpha()  # make the background transparent
        slayership = pygame.transform.scale(slayershi, (40, 40))
        meteimg1 = pygame.image.load("slayersun.png").convert_alpha()
        warperimg = pygame.image.load("warper1.png").convert_alpha()
        # sounds
        blaster = pygame.mixer.Sound("blaster-2-81267.mp3")
        collision = pygame.mixer.Sound("small-rock-break-194553.mp3")
        destroyed = pygame.mixer.Sound("rock-destroy-6409.mp3")
        speedboost = pygame.mixer.Sound("boost-100537.mp3")

        meteorspawnint = 3000  # 3 sec spawn interval
        meteors = []
        # time stuff
        lastspawntime = pygame.time.get_ticks()
        last_counter_update = pygame.time.get_ticks()
        lastwarp = pygame.time.get_ticks()
        survivaltime = 0
        spawnrate = 9000  # seconds
        minspwnint = 1000
        highscore = 0
        # upgrades
        boosters = 0
        warps = []
        pulses = []
        pulsecooldown = 5000
        warpspeed = 4.0
        gameover = False

        font = pygame.font.Font("KnightWarrior-w16n8.otf", 32)

        def areapulse():
            pulse_rect = pygame.Rect(ship.centerx, ship.centery, 80, 80)
            pulse.append(blast_rect)
            global lastpulse
            timern = pygame.time.get_ticks()
            if timern - lastpulse >= pulsecooldown:
                pulse = pygame.Rect(ship.x + ship.width // 2, ship.y, 10, 10)  # Small laser shape
                pulses.append(pulse)
                lastpulse = timern

        def warp_speed():
            size = random.randint(60, 60)
            x = random.randint(0, 800 - size)
            y = random.randint(0, 800 - size)
            warper_rect = pygame.Rect(x, y, size, size)
            warps.append(warper_rect)

        def warp_activated():
            global warpspeed
            for warper in warps:
                if ship.colliderect(warper):
                    warps.remove(warper)
                    warpspeed += 1
                    speedboost.play()

        def meteorspawn():
            side = random.randint(1, 4)
            size = random.randint(20, 60)
            speed_increment = survivaltime // 10000
            base_speed = 1 + speed_increment
            match (side):
                case 1:  # top
                    y = 0
                    x = random.randint(0, 800 - size)
                    dy = random.randint(base_speed, base_speed + 5)
                    dx = 0
                    meteor = pygame.Rect(x, y, size, size)

                case 2:  # left
                    x = 0
                    y = random.randint(0, 800 - size)
                    dx = random.randint(base_speed, base_speed + 5)
                    dy = 0
                case 3:  # right
                    x = 800 - size
                    y = random.randint(0, 800 - size)
                    dx = -random.randint(base_speed, base_speed + 5)
                    dy = 0

                case _:  # bottom
                    y = 800 - size
                    x = random.randint(0, 800 - size)
                    dy = -random.randint(base_speed, base_speed + 5)
                    dx = 0

            meteor = pygame.Rect(x, y, size, size)
            meteors.append((meteor, dx, dy))

        lasers = []
        laser_speed = -10  # Adjust speed as needed
        meteors_destroyed = []
        meteorsdestroyed = 0
        # cooldown
        lasercooldown = 1000
        lastblast = 0
        lastpulse = 0
        meteors2upgrade = 10
        holdfire_unlocked = False

        def fire_laser():
            laser_rect = pygame.Rect(ship.centerx, ship.top, 5, 15)  # Laser size
            lasers.append(laser_rect)
            global lastblast, holdfire_unlocked
            current_time = pygame.time.get_ticks()

            # Check if enough time has passed since the last shot
            if current_time - lastblast >= lasercooldown:
                laser = pygame.Rect(ship.x + ship.width // 2, ship.y, 20, 40)  # Small laser shape
                lasers.append(laser)
                lastblast = current_time

        clock = pygame.time.Clock()
        while True:

            # movement
            keys = pygame.key.get_pressed()
            for event in pygame.event.get():
                if event.type == QUIT:
                    sys.exit(0)
                elif keys[K_ESCAPE]:
                    sys.exit(0)
            if keys[K_m]:  # Go to main menu
                return False
            if keys[K_w]:
                if ship.y > 0:
                    ship = ship.move(0, -warpspeed)
            if keys[K_s]:
                if ship.y < 760:
                    ship = ship.move(0, warpspeed)
            if keys[K_d]:
                if ship.x < 760:
                    ship = ship.move(warpspeed, 0)
            if keys[K_a]:
                if ship.x > 0:
                    ship = ship.move(-warpspeed, 0)
                # get current time or time passed
            if keys[K_SPACE]:  # Spacebar to fire
                if holdfire_unlocked:
                    # Continuous shooting if upgrade is unlocked
                    fire_laser()
                    blaster.play()
                elif not holdfire_unlocked and pygame.time.get_ticks() - lastblast >= lasercooldown:
                    # Single-shot firing if no upgrade
                    fire_laser()
                    blaster.play()

            timern = pygame.time.get_ticks()
            if not gameover:
                if timern - lastspawntime > meteorspawnint:
                    meteorspawn()
                    lastspawntime = timern

                    # Update survival time counter every second
                if timern - last_counter_update >= 1000:  # Every 1000 milliseconds (1 second)
                    survivaltime += 1000
                    last_counter_update = timern

                if survivaltime % spawnrate == 0 and survivaltime > 0:
                    meteorspawnint = max(minspwnint, meteorspawnint - 500)
                if timern - lastwarp > 10000:
                    warp_speed()
                    lastwarp = timern

                warp_activated()

            for i in range(len(meteors)):
                meteor_rect, dx, dy = meteors[i]
                meteor_rect.x += dx  # Move the meteor
                meteor_rect.y += dy
                # Update laser positions and check for collisions
            for laser in lasers[:]:  # Copy list to safely remove items
                laser.y += laser_speed
                # Remove laser if it goes off-screen
                if laser.bottom < 0:
                    lasers.remove(laser)

                # Check for collision with meteors
                for meteor in meteors[:]:  # Copy list to safely remove items
                    if laser.colliderect(meteor[0]):
                        meteors.remove(meteor)
                        lasers.remove(laser)
                        destroyed.play()
                        meteorsdestroyed += 1  # Increment meteors destroyed
                        break
                # Check if laser upgrade should be unlocked
                if meteorsdestroyed >= meteors2upgrade:
                    holdfire_unlocked = True  # Allow holding space to shoot
                    laser_speed = -15  # Increase laser speed (optional upgrade)

            background3()

            screen.blit(slayership, ship)
            for meteor_rect, dx, dy in meteors:
                # Scale the meteor image according to the current meteor size
                scaled_meteor_image = pygame.transform.scale(meteimg1, (meteor_rect.width, meteor_rect.height))
                # Blit the scaled meteor image at the meteor's position
                screen.blit(scaled_meteor_image, meteor_rect)
            for meteor in meteors:
                if ship.colliderect(meteor[0]):
                    collision.play()
                    gameover = True
            for laser in lasers:
                pygame.draw.rect(screen, (66, 152, 250), laser)
            for warper_rect in warps:
                scaled_warp = pygame.transform.scale(warperimg, (warper_rect.width, warper_rect.height))
                screen.blit(scaled_warp, warper_rect)

            if gameover:
                if meteorsdestroyed >= 30:
                        result = "win"
                else:
                        result = "lose"

                if not gameover_screen(result):
                        return
            if not gameover:
                timer_text = font.render("Survival Time: " + str(survivaltime // 1000) + "s", True, (255, 255, 255))
                screen.blit(timer_text, (10, 10))
                Score = font.render(f"Score: {meteorsdestroyed} ", True, (255, 0, 0))
                screen.blit(Score, (10, 50))
                menu = font.render("Press M to return to the menu", True,(255,255,0))
                screen.blit(menu,(10,85))
                if meteorsdestroyed >= 30:
                    result= "win"
                    gameover = True


            else:
                # Display final survival time
                final_time_text = font.render("Game Over! Final Time: " + str(survivaltime // 1000) + "s", True,
                                              (255, 0, 0))
                screen.blit(final_time_text, (200, 400))

                if survivaltime > highscore:
                    highscore = survivaltime

                # Display longest survival time
                highscore_text = font.render("Longest Survival Time: " + str(highscore // 1000) + "s", True,
                                             (255, 255, 0))
                screen.blit(highscore_text, (200, 450))

                pygame.display.flip()
                pygame.time.wait(3000)
                gameover = False
                ship.topleft = (380, 380)
                meteors.clear()
                lastspawntime = pygame.time.get_ticks()
                survivaltime = 0
                meteorspawnint = 3000 # reset spawn rate
                meteorsdestroyed = 0  # reset upgrade
                holdfire_unlocked = False  # reset holdfire upgrade
                warpspeed = 4
                warps.clear()
                lastwarp = pygame.time.get_ticks()

            pygame.display.flip()
            clock.tick(60)





def main_menu():
    while True:
        keys = pygame.key.get_pressed()
        for event in pygame.event.get():
            if event.type == QUIT:
                sys.exit(0)
            elif keys[K_ESCAPE]:
                sys.exit(0)
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:  # Start game when ENTER is pressed
                    start_slayer()
                if event.key == pygame.K_1: #start ez mode
                    easy()
        menu_mouse_pos = pygame.mouse.get_pos()
        background()
        font = pygame.font.Font("KnightWarrior-w16n8.otf", 60)
        fontsmall = pygame.font.Font("KnightWarrior-w16n8.otf",15)
        title_text = font.render("Astral Slayer", True, (255,255,255))
        screen.blit(title_text, (screen.get_width()//2 - title_text.get_width()//2, screen.get_height()//4))
        play_text = font.render("Press ENTER for Endless", True, (255,255,255))
        easy_text = font.render("Press 1 for Easy Mode", True, (255,255,255))
        screen.blit(easy_text, (screen.get_width()//2 - easy_text.get_width()//2,screen.get_width()/1.5))
        instructions = fontsmall.render("Use WASD to move around and dodgde meteors! \nDrive into warpers for a lightspeed boost.\n Destroy meteors for upgrades!", True,(0,255,0))
        screen.blit(instructions,(10,10))

        screen.blit(play_text, (screen.get_width()//2 - play_text.get_width()//2, screen.get_width()//2))


        pygame.display.flip()
        clock.tick(FPS)

main_menu()
#game
