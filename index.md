![](https://raw.githubusercontent.com/jierueichang/astrochicken/master/stars.png)
In his book *Disturbing the Universe* (1979), Freeman Dyson explores the concept of an *Astrochicken*, a von Neumann machine that would replicate itself by harvesting materials from asteroids. Such a machine could be launched from Earth and explore space much more efficiently than with conventional spacecraft. This game is based on this concept.

The objective of this game is to colonize as many asteroids as possible by landing various types of robotic 'chickens' on them. The player controls the main *Astrochicken*, which stays in space.

There are three kinds of chickens:

| Types         | Purpose                | Key | Image |
| ------------- | ---------------------- | --- |:---:|
| Astrochicken  | Creates Chickens       | ~   | ![](https://raw.githubusercontent.com/jierueichang/astrochicken/master/astrochicken.png)
| Colonist      | Colonizes asteroids    | `c` | ![](https://raw.githubusercontent.com/jierueichang/astrochicken/master/colonistchicken.png)
| Architect     | Constructs Greenhouses | `a` |![](https://raw.githubusercontent.com/jierueichang/astrochicken/master/explorerchicken.png)

The movement of the Astrochicken is controlled by the arrow keys.

Players need to expend *resources* to create Colonists and Architects. To obtain resources a player will need to mine asteroids. This is done by skimming over the orange, small asteroids while pressing and releasing the `m` key. The Astrochicken cannot collide with any other type of asteroid.

Source code:
```
import pygame
from pygame.locals import *
pygame.init()
dim = [500,500]
screen = pygame.display.set_mode((500,500),RESIZABLE)
pygame.display.set_caption('Astrochicken')
ticker = pygame.time.Clock()
#asteroids = pygame.transform.scale(pygame.image.load('asteroids.png'),(3072,1508))
import random

'''IMAGES'''
asteroid1 = pygame.transform.scale(pygame.image.load('asteroid1.png'),(440,374))
asteroid2 = pygame.transform.scale(pygame.image.load('asteroid2.png'),(330,253))
asteroid3 = pygame.transform.scale(pygame.image.load('asteroid3.png'),(242,242))
asteroid4 = pygame.transform.scale(pygame.image.load('asteroid4.png'),(121,110))
stars = pygame.image.load('stars.png')
astrochicken = pygame.transform.scale(pygame.image.load('astrochicken.png'),(240/4,165/4))
architectchicken = pygame.transform.scale(pygame.image.load('explorerchicken.png'),(210/4,165/4))
colonistchicken = pygame.transform.scale(pygame.image.load('colonistchicken.png'),(150/4,240/4))
greenhouseplant = pygame.transform.scale(pygame.image.load('greenhouse.png'),(495/4,450/4))

backx = 0
backy = 0

myfont = pygame.font.Font(None,80)
class asteroid():
    def __init__(self,x,y):
        self.x = x
        self.y = y
        self.types = random.choice([asteroid1,asteroid2,asteroid3,asteroid4])
        self.rect = self.types.get_rect()
        self.rect.x = self.x
        self.rect.y = self.y
        self.timer = 0
        self.mined = 100
    def step(self,xspeed,yspeed):
        self.x += xspeed
        self.y += yspeed
        self.rect.x = self.x
        self.rect.y = self.y
    def render(self,screen):
        screen.blit(self.types,(self.x,self.y))
    def growgreenhouse(self,colonists,architects,greenhouses):
        for x in colonists:
            if self.rect.colliderect(x.rect):
                for y in architects:
                    if y.rect.colliderect(self.rect):
                        self.timer += 1
        if self.timer >= 100:
            greenhouses.append(greenhouse(random.randint(self.rect.x,self.rect.x+self.rect.width),random.randint(self.rect.y,self.rect.y+self.rect.height)))
            print 'yay!'
            self.timer = 0
class chicken():
    def __init__(self,xspeed,yspeed):
        global material, dim
        self.x = 230
        self.y = 230
        self.xs = xspeed
        self.ys = yspeed
        self.hit = False
        self.timer = 0
        self.rect = astrochicken.get_rect()
        self.rect.x = self.x
        self.rect.y = self.y
    def step(self,xpeed,yspeed):
        global material, colonists, architects
        self.x -= self.xs
        self.y -= self.ys
        self.rect.x = self.x
        self.rect.y = self.y            
        for x in asteroids:
            if x.rect.colliderect(self.rect) and x.type != asteroid4:
                self.hit = True
        if self.timer >= 5:
            self.planted = True
        self.x += xspeed
        self.y += yspeed
    def render(self,screen):
        screen.blit(colonistchicken,(self.x,self.y))
class colonist():
    def __init__(self,xspeed,yspeed):
        global material, dim
        self.x = dim[0]/2-20
        self.y = dim[1]/2-20
        self.xs = xspeed
        self.ys = yspeed
        self.planted = False
        self.hit = False
        self.timer = 0
        self.rect = colonistchicken.get_rect()
        self.rect.x = self.x
        self.rect.y = self.y
    def step(self,xpeed,yspeed):
        global material
        if not self.planted:
            self.x -= self.xs
            self.y -= self.ys
            self.rect.x = self.x
            self.rect.y = self.y            
            for x in asteroids:
                if x.rect.colliderect(self.rect):
                    self.hit = True
            if self.hit:
                self.timer += 1
            if self.timer >= 5:
                self.planted = True
        else:
            self.timer += 1
            if self.timer >= 100:
                material += 5
                self.timer = 0
            
        self.x += xspeed
        self.y += yspeed
    def render(self,screen):
        screen.blit(colonistchicken,(self.x,self.y))
class architect():
    def __init__(self,xspeed,yspeed):
        global material, dim
        self.x = dim[0]/2-20
        self.y = dim[1]/2-20
        self.xs = xspeed
        self.ys = yspeed
        self.planted = False
        self.timer = 0
        self.rect = architectchicken.get_rect()
        self.rect.x = self.x
        self.rect.y = self.y
    def step(self,xpeed,yspeed):
        global material
        if not self.planted:
            self.rect.x = self.x
            self.rect.y = self.y
            self.x -= self.xs
            self.y -= self.ys
            for x in asteroids:
                if x.rect.colliderect(self.rect):
                    self.planted = True
        else:
            self.timer += 1
            if self.timer >= 150:
                self.planted = False
                material += 20
                self.timer = 0
        self.x += xspeed
        self.y += yspeed
    def render(self,screen):
        screen.blit(architectchicken,(self.x,self.y))
class greenhouse():
    def __init__(self,x,y):
        self.x = x
        self.y = y
        self.timer = 0
        self.x += xspeed
        self.y += yspeed
        self.rect = greenhouseplant.get_rect
    def step(self,screen,xspeed,yspeed):
        global material, colonists
        self.x += xspeed
        self.y += yspeed
        self.timer += 1
        screen.blit(greenhouseplant,(self.x,self.y))
        if self.timer > 300:
            colonists.append(random.randint(self.rect.x,self.rect.x+self.rect.width),random.randint(self.rect.y,self.rect.y+self.rect.height))   
        if self.timer > 100:
            material += 100
            self.timer = 0
def run():
    global screen,backx,backy,xspeed,yspeed,asteroids,material,dim
    while True:
        screen.fill((0,0,0))
        #screen.blit(asteroids,(x,y))
        screen.blit(stars,(backx,backy))
        backx+=xspeed/10
        backy+=yspeed/10
        for item in asteroids:
            item.step(xspeed,yspeed)
            item.render(screen)
            item.growgreenhouse(colonists,architects,greenhouses)
            if item.rect.colliderect(pygame.rect.Rect(dim[0]/2-20,dim[1]/2-20,40,40)):
                if not item.types == asteroid4:
                    return
              
        for item in architects:
            item.step(xspeed,yspeed)
            item.render(screen)
        for item in colonists:
            item.step(xspeed,yspeed)
            item.render(screen)
        for item in greenhouses:
            item.step(screen,xspeed,yspeed)
        for item in colonists:
            item.render(screen)  
        screen.blit(astrochicken,(dim[0]/2-20,dim[1]/2-20))
        screen.blit(myfont.render(str(material),1,(100,100,255)),(20,20))
        for event in pygame.event.get():
            if event.type == QUIT:
                pygame.quit()
            if event.type == KEYDOWN:
                if event.key == K_UP:
                    yspeed +=2
                if event.key == K_DOWN:
                    yspeed -=2
                if event.key == K_RIGHT:
                    xspeed -=2
                if event.key == K_LEFT:
                    xspeed += 2
                if event.key == K_c and material >= 20:
                    colonists.append(colonist(xspeed,yspeed))
                    material -= 20
                if event.key == K_a and material >= 40:
                    architects.append(architect(xspeed,yspeed))
                    material -= 40
                if event.key == K_m:
                    for item in range(len(asteroids)):
                        x = asteroids[item]
                        if x.types == asteroid4 and x.rect.colliderect(pygame.rect.Rect(dim[0]/2-20,dim[1]/2-20,40,40)) and (xspeed != 0 or yspeed != 0):
                            material += 2
                            x.mined -= 1
                            if x.mined <= 1:
                                asteroids.pop(item)
                            break
            if event.type == VIDEORESIZE:
                for a in asteroids:
                    a.x += (event.w-dim[0])/2
                    a.y += (event.h-dim[1])/2
                    a.rect.x += (event.w-dim[0])/2
                    a.rect.y += (event.h-dim[1])/2
                for x in colonists:
                    x.x+=(event.w-dim[0])/2
                    x.y+=(event.h-dim[1])/2
                for y in architects:
                    y.x+=(event.w-dim[0])/2
                    y.y+=(event.h-dim[1])/2
                for z in greenhouses:
                    z.x+=(event.w-dim[0])/2
                    x.y+=(event.h-dim[1])/2
                dim = [event.w,event.h]
                screen = pygame.display.set_mode((event.w,event.h),RESIZABLE)
        pygame.display.update()
        ticker.tick(20)

def main():
    global xspeed,yspeed,asteroids,architects,colonists,greenhouses,material
    while True:
        material = 50
        architects = []
        colonists = []
        greenhouses = []
        xspeed = 0
        yspeed = 0
        asteroids = []
        for x in range(0,100):
            asteroids.append(asteroid(random.randint(-4000,4000),random.randint(-4000,4000)))    
        asteroids.append(asteroid(150,300))         
        run()

if __name__ == '__main__':
    main()
```
