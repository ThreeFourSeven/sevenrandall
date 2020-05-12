---
layout: post
title: "Intro to Python Game Development with Pygame"
date: 2020-05-12
---

## Introduction to Python Game Development with Pygame 

All you need to get started is python and pip, any text editor will work. This blog will cover several topics
such as. 

* setting up pygame
* using pygame to create a display
* drawing with pygame
* detect collision between various shapes
* design a simple Entity Component System     
* finally will use the covered topics to make a simple game

This tutorial assumes that you are familiar with python classes and inheritance. 

### Part 1 Setup

First step is to open a terminal in an empty folder and type.

> pip install pygame

After pygame is installed successfully open a text editor of your choosing and include.

> import pygame    

Now to make the display, pygame first requires you to call.

> pygame.display.init()

After that is called we can create the display with.

> pygame.display.set_mode((width, height), flags, depth)

Width and Height correspond to the dimensions of the window.
Flags is a bitfeild that stores a number of useful settings such as.

* FULLSCREEN
* DOUBLEBUF
* RESIZABLE
* SCALED
* NOFRAME
* HWSURFACE

Finally depth is the number of bits to use per color. 

> def create_display(width, height, title="pygame-display"):
>
>   pygame.display.init()
>
>   display = pygame.display.set_mode((width, height), 0, depth)
>
>   pygame.display.set_caption(title)
>
>   return display

### Part 2 Using Pygame
...contents...

### Part 3 Making a Simple Game
...contents...