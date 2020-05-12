---
layout: post
title: "Intro to Python Game Dev with Pygame Part 1"
date: 2020-05-12
---

## Intro to Python Game Dev with Pygame Part 1

This tutorial assumes that you are familiar with python, and have **[PyCharm](https://www.jetbrains.com/pycharm/)** already installed. This and following parts will cover.

- Setting up pygame.
- Using pygame to create a display.
- Drawing with pygame.
- Detecting collision between various shapes.
- An Entity Component System.    
- Making a game.

The First step is to install pygame, PyCharm makes this easy for us. Simply create a new python containing this.

```python
import pygame
```

As long as your project's python enviornment is configured properly you should be able to hover over the error and click **install package pygame**. 
Now that the dependencies are delt with it is time we establish the structure our game will have.  

```python
running = True
init()
while running:
  poll_input()
  update()
  clear_input()
  clear()
  render()
  swap_frame()
```

> init()

Will contain operations that can be performed once, or before game launch. Some examples would be asset loading or sometimes world generation. 

> poll_input()

Will get the current keyboard and mouse events from the display.

> update()

Will contain operations that need to performed every frame. Examples would be responding to user input, moving characters, and detecting collisions.

> clear_input()

Will clear all stored keyboard and mouse events

> clear()

Will clear the current frame to a single color.

> render()

Will contain all pygame draw operations.

> swap_frame()

Will Swap the old frame for the current frame.

---

So from this our display will need to.

  - Create a frame with a given title, width and height.
  - Clear a frame to a single color.
  - Swap frames.
  - Close on user input.

Now to create the display, pygame first requires you to call.

```python
pygame.display.init()
```

After that is called we can create the display with.

```python
pygame.display.set_mode((width, height), flags, depth)
```
**Width** and **Height** correspond to the dimensions of the display. **Flags** is a [bitfeild](https://wiki.python.org/moin/BitManipulation) that stores several useful boolean values such as.

- FULLSCREEN
- DOUBLEBUF
- RESIZABLE
- SCALED
- NOFRAME
- HWSURFACE

Finally depth is the number of bits to use per color. Putting this all together in a function we get.

```python
def create_display(width, height, title="pygame-display"):
  pygame.display.init()
  display = pygame.display.set_mode((width, height), 0, 32)
  pygame.display.set_caption(title)
  return display
```

If you call this nothing will happen, and thats because the display is created for a moment and then instantly closed. This may appear like a problem but actually everything is working correctly, we just need to have some process either wait or use the created window. To start well create a class which will contain all the methods covered above.

```python
class Game:
  def __init__(self):
    self.running = True
    self.keys_clicked = {}
    self.buttons_clicked = {}
    self.keys_down = []
    self.buttons_down = []
    self.key_mods = 0
    self.mouse_position = [0, 0]
    self.previous_mouse_position = [0, 0]
    self.mouse_velocity = [0, 0]
    self.display = None

  def init(self, width=1600, height=900):
    self.display = create_display(width, height)

  def poll_input(self):
    pass

  def update(self):
    pass

  def clear_input(self):
    pass

  def clear(self):
    pass

  def render(self):
    pass

  def swap_frame(self):
    pass

  def run(self):
    self.init()
    while self.running:
      self.poll_input()
      self.update()
      self.clear_input()
      self.clear()
      self.render()
      self.swap_frame()
```

These will store all input events coming from our display. 

```python
self.keys_clicked = {}
self.buttons_clicked = {}
self.keys_down = []
self.buttons_down = []
self.key_mods = 0
self.mouse_position = [0, 0]
self.previous_mouse_position = [0, 0]
self.mouse_velocity = [0, 0]
```

Example usage.

```python
#Game.update()
def update(self):
  if pygame.K_SPACE in self.keys_clicked and self.keys_clicked[pygame.K_SPACE]:
    print("Clicked the space bar!")

  if self.buttons_down[0] and self.buttons_down[2]:
    print("The left and right mouse button are held down!")

  if 0 in self.buttons_clicked and self.buttons_clicked[0]:
      print("The left mouse button was clicked!")

  if self.keys_down[pygame.K_SPACE] and self.key_mods & pygame.KMOD_SHIFT:
    print("The space bar and shift are held down!")

  print("The mouse is located at", self.mouse_position, "with a velocity of", self.mouse_velocity)
```

In order to have the above information we gather the events from pygame every frame.

```python
# Game.poll_input()
def poll_input(self):
  for event in pygame.event.get():
    if event.type == pygame.QUIT:
      self.running = False
    elif event.type == pygame.KEYDOWN:
      self.keys_clicked[event.key] = True
    elif event.type == pygame.MOUSEBUTTONDOWN:
      self.buttons_clicked[event.button-1] = True

  self.keys_down = pygame.key.get_pressed()
  self.buttons_down = pygame.mouse.get_pressed()
  self.key_mods = pygame.key.get_mods()

  tuple_mouse_pos = pygame.mouse.get_pos()
  self.mouse_position[0] = tuple_mouse_pos[0]
  self.mouse_position[1] = tuple_mouse_pos[1]

  if self.previous_mouse_position[0] != self.mouse_position[0] and self.previous_mouse_position[1] != self.mouse_position[1]:
    self.mouse_velocity[0] = self.mouse_position[0] - self.previous_mouse_position[0]
    self.mouse_velocity[1] = self.mouse_position[1] - self.previous_mouse_position[1]
    self.previous_mouse_position = self.mouse_position.copy()
```

You may be confused why there is a -1, well for some reason event.button returns a value from 1 to 3, while pygame.mouse.get_pressed() returns a value from 0 to 2. 

```python
self.buttons_clicked[event.button-1] = True
```

To clear the stored events we only need to clear the dictionaries as the lists are reset every frame.

```python
#Game.clear_input()
def clear_input(self):
  self.keys_clicked.clear()
  self.buttons_clicked.clear()
```

Now we have the display opening and responding to input events with. 

```python
Game().run()
```

Next part will cover how to render objects to the screen, and detect when objects are colliding. 