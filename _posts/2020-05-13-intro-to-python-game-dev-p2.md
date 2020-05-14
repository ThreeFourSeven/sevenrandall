---
layout: post
title: "Intro to Python Game Dev with Pygame Part 2"
date: 2020-05-13
comments: true
---

## Intro to Python Game Dev with Pygame Part 2

Time to start drawing. To get started we'll need to implement the `clear` and `swap_frame` methods covered in the last part. Just to reiterate, the clear method will set the current frame to a single color, and the swap_frame method will swap the display's frame with the current frame we have drawn onto. Pygame makes this easy.  

```python
#Game.clear()
def clear(self):
    # sets the color of every pixel in the current frame to black 
    self.display.fill(pygame.Color(0x000000ff))
```

This uses the display instance returned by `create_display` and fills the current frame with a **[pygame.Color](https://www.pygame.org/docs/ref/color.html)**. To swap the current frame with the new one we have `pygame.display.flip()`, but this wont work for drawing text, or textures. Pygame requires you to call `pygame.display.update()` for blending text and textures together which gets us.

```python
#Game.swap_frame()
def swap_frame(self):
    pygame.display.update()
    pygame.display.flip()
```

One way to improve the performance would be to make a boolean flag which is only true when a texture or text is drawn that way you wont have any unecesary update calls. Before we go any further we also need to include this line in `init()` to initialize pygame's default font.

```python
#Game.init()
pygame.font.init()
```

Now that pygame and the current frame are ready to be drawn onto we can write some draw methods. While these aren't required, they will make life easier as you wont need to pass `self.display` or deal with merging frames using `display.blit`. 

```python
#Game
def draw_circle(self, x: int, y: int, radius: int, color: pygame.Color):
    pygame.draw.circle(self.display, color, (x, y), radius)

def draw_box(self, x: int, y: int, width: int, height: int, color: pygame.Color, bool: center = False):
 # if centered then will use the box's size to offset the position
    pos = (x, y) if not center else (x - width//2, y - height//2)
    pygame.draw.rect(self.display, color, (pos[0], pos[1], width, height))

def draw_line(self, sx: int, sy: int, ex: int, ey: int, color: pygame.Color, thickness: int = 1):
    pygame.draw.line(self.display, color, (sx, sy), (ex, ey), thickness)

def draw_text(self, txt: str, x: int, y: int, size: int, color: pygame.Color, center: bool = False):
    # creates a default font atlas texture given the size
    font = pygame.font.Font(None, size)
    # returns a frame with our colored text   
    text_frame = font.render(txt, True, color)
    # if centered then will use the text_frame's size to calulate the offset
    pos = (x, y) if not center else (x - text_frame.get_width()//2, y - text_frame.get_height()//2)
    # merges the current frame with the text frame, make sure pygame.display.update is called
    self.display.blit(text_frame, pos)
```

Now that the basic drawing mehods are done its time to implement texture loading, as our game will require more than just text and colored shapes. First step is to make a texture loading method which will read an image file and return a texture object we can use to draw said image. See [pygame.image.load](https://www.pygame.org/docs/ref/image.html#pygame.image.load) for supported image file types.

```python
# Top of main file
import os
def load_texture(path):
    # Will get absolute path to the folder that contains your main file
    full_path = os.path.dirname(os.path.abspath(__file__))
    # Will load image file and return a frame
    return pygame.image.load(full_path+"/"+path)

# Example usage if your project dir looks like: main.py, texture.png
load_texture("texture.png")
```

To draw a texture.

```python
#Game.draw_texture
def draw_texture(self, x: int, y: int, texture_frame, center: bool = False):
    pos = (x, y) if not center else (x - texture_frame.get_width()//2, y - texture_frame.get_height()//2)
    self.display.blit(texture_frame, pos)
```

Now you should be able to test out these draw methods by calling them in `render()`, but first lets record the width and height of our window to make positioning objects easier.

```python
# In Game.__init__()
self.width = 0
self.height = 0

# In Game.init()
self.width = width
self.height = height
```

To draw some objects.

```python
#Game.__init__()
self.texture = None

#Game.init()
self.texture = load_texture("texture.png")

#Game.render()
def render():
    # The order of the draw calls controls the depth of the objects
    # For example draw the background first, objects second, and leave the user interface to last
    # Draws a red box at the mouse cursor
    self.draw_box(self.mouse_position[0], self.mouse_position[1], 32, 32, pygame.Color(0xff0000ff))
    # Draws some centered white text at the center of the display 
    self.draw_text("Hello pygame!", self.width//2, self.height//2, 20, pygame.Color(0xffffffff), True)
    # Will draw a 3x3 grid with your texture 
    for i in range(0, 4):
        for j in range(0, 4):
            self.draw_texture(j*self.texture.get_width(), i*self.texture.get_height(), self.texture, True)
```

Now that all the draw methods required to make a game are done, how do we detect when an object is overlapping or colliding with another? We can write a few methods to detect collision between various shapes, such as lines, circles, boxes, and much more. I recommend you do some research into collision detection if the following do not work for your game. 

```python
#Above Game
def point_circle(point: (int , int), circle: (int, int, int)):
    return abs(point[0] - circle[0]) < circle[2] and abs(point[1] - circle[1]) < circle[2]

def circle_circle(circle_a: (int, int, int), circle_b: (int, int, int)):
    distance = (cirlce_a[0] - circle_b[0], cirlce_a[1] - circle_b[1])
    radius = circle_a[2] + circle_b[2]
    return distance[0]**2 + distance[1]**2 <= radius

def box_box(box_a: (int, int, int, int), box_b: (int, int, int, int)):
    collision_x = box_a[0] + box_a[2] >= box_b[0] and box_b[0] + box_b[2] >= box_a[0]
    collision_y = box_a[1] + box_a[3] >= box_b[3] and box_b[1] + box_b[3] >= box_a[1]
    return collision_x and collision_y

def circle_box(circle: (int, int, int), box: (int, int, int, int)):
    closest_x = max(min(circle[0], box[0] + box[2]//2), box[0] - box[2]//2)
    closest_y = max(min(circle[1], box[1] + box[3]//2), box[1] - box[3]//2)
    distance_x = circle[0] - closest_x
    distance_y = circle[1] - closest_y
    return distance_x ** 2 + distance_y ** 2 <= circle[2] ** 2

def point_box(point: (int, int), box: (int, int, int, int)):
    return box_box((point[0], point[1], 0, 0), box)
```

A simple example using both drawing and collision detection would be.

```python
# In Game.render()
box = (64, 64, 32, 32)
point = (self.mouse_position[0], self.mouse_position[1])
# If the mouse is inside the box the box will be green else it will be red
if point_box(point, box):
    self.draw_box(box[0], box[1], box[2], box[3], pygame.Color(0x00ff00ff))
else:
    self.draw_box(box[0], box[1], box[2], box[3], pygame.Color(0xff0000ff))

box1 = (self.width//2, 128, 64, 16)
if point_box(point, box1) and self.is_button_down(0):
    self.draw_text("<- Selected!", self.width//2+64, 128, 20, pygame.Color(0xffffffff))
```

With this you should start to get an idea of how a simple game could be made, but you'll find as you add more to `Game` and make other classes that there is something repetitive about creating an object, describing its location and size, drawing it, then responding to either user or collision events. So we can do better! Next part will be focused on designing a simple Entity Component System to fix this issue.