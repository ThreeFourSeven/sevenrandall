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

Will get the current keyboard and mouse events from the window.

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
    self.keys = {}
    self.buttons = {}
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

These dictionarys will be filled with pressed keys and buttons and then cleared after use.

```python
self.keys = {}
self.buttons = {}
```

Example usage.

```python
if self.keys[pygame.K_SPACE]:
  print("Pressed the space bar!")

if self.buttons[0] and self.buttons[2]:
  print("Pressed the left and right mouse button!")
```

To store the key and button events from pygame.

```python
# Game.poll_input()
def poll_input(self):
  for event in pygame.event.get():
    if event.type == pygame.QUIT:
      self.running = False
    elif event.type == pygame.KEYDOWN:
      self.keys[event.key] = True
    elif event.type == pygame.MOUSEBUTTONDOWN:
      self.buttons[event.button] = True
```

Clearing the stored events.

```python
#Game.clear_input()
def clear_input(self):
  self.keys.clear()
  self.buttons.clear()
```

Now have the display opening and responding to input events with. 

```python
Game().run()
```

Next time well cover how to render objects to the screen, and detect when objects are colliding. 