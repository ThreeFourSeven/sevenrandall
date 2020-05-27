---
layout: post
title: "Intro to Python Game Dev with Pygame Part 3"
date: 2020-05-14
comments: true
---

## Intro to Python Game Dev with Pygame Part 3

Lets look at how an Entity Component System can reduce redundant code while making a game. An entity can be considered a character or object in our game, from a wall to a player. Components make up the properties and logic of an entity, this can be done in a number of ways. 

```python
class Entity:
    _nextID = 0
    def __init__(self):
        self.position = [0 ,0]
        self.components = {}
        self.id = self._nextID
        self._nextID += 1
    
    def add_component(self, component):
        component.entity = self
        self.components[component.component_type] = component
    
    def update(self, game):
        for component in self.components.values():
            component.update(game)

    def draw(self, game):
        for component in self.components.values():
            component.draw(game)

class Component:
    def __init__(self, component_type):
        self.component_type = component_type
        self.entity = None
    
    def update(self, game):
        pass
    
    def draw(self, game):
        pass

```

The idea is to have every object in our game represented by an entity, with each entity containing a dictionary of components. A component being an abstract class that has a component type and a reference to the entity that stores it. This allows us to add any class that extends component to an entity, which reduces redudant code. Lets say you wanted to make a entity that acted as a spike that would remove any other entity that collided with it. You can make a component that would deal with the collision and removal. So to get an entity to behave as a spike just add the component. This may be difficult to follow or visualize so here are some example components.

```python
# Each component has a unique type to prevent an entity from having two of one component 
class ComponentType:
    Shape_ = 0
    Controller_ = 1
    Drift_ = 2 

# Will regularly move an entity in a given direction and speed 
class Drift(Component):
    def __init__(self, direciton, speed:int = 1)
        # Sets the Drift component to be a ComponentType.Drift_ component
        super().__init__(ComponentType.Drift_)
        self.direction = direction
        self.speed = speed
    
    def update(self, game):
        self.entity.position[0] += self.direction[0]*self.speed
        self.entity.position[1] += self.direction[1]*self.speed

# Will allow you to move the entity arround with either the arrow keys or WASD
class Controller(Component):
    def __init__(self, speed):
        super().__init__(ComponentType.Controller_)
        self.speed = speed

    def update(self, game):
        if game.is_key_down(pygame.K_w) or game.is_key_down(pygame.K_UP):
            self.entity.position[1] -= self.speed
        if game.is_key_down(pygame.K_s) or game.is_key_down(pygame.K_DOWN):
            self.entity.position[1] += self.speed
        if game.is_key_down(pygame.K_a) or game.is_key_down(pygame.K_LEFT):
            self.entity.position[0] -= self.speed
        if game.is_key_down(pygame.K_d) or game.is_key_down(pygame.K_RIGHT):
            self.entity.position[0] += self.speed

# Types of shapes for the shape component 
class ShapeType:
    Circle_ = 0
    Box_ = 1

def default_shape_on_collide(entity_a, entity_b):
    pass

# A shape component will draw the shape or texture and throw a callback when a collision is detected
class Shape(Component):
    def __init__(self, shape, shape_size, color):
        super().__init__(ComponentType.Shape_)
        self.shape = shape
        # For a circle [radius] for a box [width, height]
        self.shape_size = shape_size
        # A shape will call this method whenever it collides with another shape
        self.on_collide = default_shape_on_collide
        # Ignores collision from these entities  
        self.ignored_entities = set()
        # By default no texture is drawn, if a texture is set than it will replace the colored shape
        self.texture = None
        self.color = color

    # This method looks ugly but it is just using the collision methods from last part
    def collides(self, other):
        if other.entity.id in self.ignored_entities:
            return
        if self.shape == ShapeType.Circle_:
            # Circle v. Circle
            if other.shape == ShapeType.Circle_ and circle_circle_(
                    (self.entity.position[0], self.entity.position[1], self.entity.scale * self.shape_size[0]),
                    (other.entity.position[0], other.entity.position[1], other.entity.scale * other.shape_size[0])):
                self.on_collide(self.entity, other.entity)
            # Circle v. Box
            elif other.shape == ShapeType.Box_ and circle_box(
                    (self.entity.position[0], self.entity.position[1], self.entity.scale * self.shape_size[0]), (
                            other.entity.position[0], other.entity.position[1],
                            other.entity.scale * other.shape_size[0],
                            other.entity.scale * other.shape_size[1])):
                self.on_collide(self.entity, other.entity)
        elif self.shape == ShapeType.Box_:
            # Box v. Circle
            if other.shape == ShapeType.Circle_ and circle_box(
                    (other.entity.position[0], other.entity.position[1], other.entity.scale * other.shape_size[0]), (
                            self.entity.position[0], self.entity.position[1], self.entity.scale * self.shape_size[0],
                            self.entity.scale * self.shape_size[1])):
                self.on_collide(self.entity, other.entity)
            # Box v. Box
            elif other.shape == ShapeType.Box_ and box_in_box((
                    self.entity.position[0], self.entity.position[1], self.entity.scale * self.shape_size[0],
                    self.entity.scale * self.shape_size[1]), (
                    other.entity.position[0], other.entity.position[1], other.entity.scale * other.shape_size[0],
                    other.entity.scale * other.shape_size[1])):
                self.on_collide(self.entity, other.entity)

    def draw(self, game):
        # If a texture has not been set then draw a colored shape
        if self.texture is None:
            if self.shape == ShapeType.Circle_:
                game.draw_circle(self.entity.position[0], self.entity.position[1],
                                 self.entity.scale * self.shape_size[0],
                                 self.color)
            elif self.shape == ShapeType.Box_:
                game.draw_box(self.entity.position[0], self.entity.position[1], self.entity.scale * self.shape_size[0],
                              self.entity.scale * self.shape_size[1], self.color, True)
        else:
            game.draw_texture(self.entity.position[0], self.entity.position[1], self.texture, True)
```

To allow entities to access each other I choose to use a global dictionary to store active entities. 

```python
# Global map of all entities 
ENTITIES = {}
# Global list to contain entity ids of entities that should be removed
TO_REMOVE = []

def add_entity(entity):
    ENTITIES[entity.id] = entity

# To prevent removing an entity during iteration
def remove_entity(entity):
    if entity.id in ENTITIES:
        TO_REMOVE.append(entity.id)

# Game.update()
def update(self)
    # Update all entities
    for entity in ENTITIES.values():
        entity.update(self)
    # Shape collision detection
    for e_a in ENTITIES.values():
        for e_b in ENTITIES.values():
            if e_a.id != e_b.id:
                # If an entity doesn't contain a shape then python will throw a KeyError
                try:
                    e_a.components[ComponentType.Shape_].colliding_with(e_b.components[ComponentType.Shape_])
                except KeyError:
                    pass
    # Removes entities from map that were requested to be removed
    for eid in TO_REMOVE:
        ENTITIES.pop(eid)
    TO_REMOVE.clear()

# Game.render()
def draw(self):
    # Draw all entities
     for entity in ENTITIES.values():
        entity.draw(self)
```

If you are confused and have no idea how any of this will make life easier thats okay, I urge you to bare with me and look at this example. Lets say you wanted to clone a mobile game about a bird that tries to not fall off the map or run into green poles. Then we can make a pole and bird entity that each contain their own logic.

```python
DISPLAY_WIDTH = 1600
DISPLAY_HEIGHT = 900

POLE_WIDTH = 64
POLE_HEIGHT = 150
SPAWN_OFFSET = 125
class Pole(Entity):
    def __init__(self, start_x, start_y):
        super().__init__()
        self.start_x = start_x
        self.start_y = start_y
        # Will make the pole slowly move to the left
        self.add_component(Drift([-1, 0]))
        # Green box
        self.add_component(Shape(ShapeType.Box_, [POLE_WIDTH, POLE_HEIGHT], pygame.Color(0x00ff00ff)))

    def update(self, game):
        super().update(game)
        # If pole goes off left side of screen respawn to starting position
        if self.position[0] <= -POLE_WIDTH:
            self.position[0] = DISPLAY_WIDTH+self.start_x

    def draw(self, game):
        super().draw(game)

# Game.init()
# adds 20 poles to the bottom and top of our display but off to the right of the screen
for i in range(0, 10):
    add_entity(Pole(DISPLAY_WIDTH+i*(POLE_WIDTH+SPAWN_OFFSET), POLE_HEIGHT//2))
    add_entity(Pole(DISPLAY_WIDTH+i*(POLE_WIDTH+SPAWN_OFFSET), DISPLAY_HEIGHT-POLE_HEIGHT//2))

# None of the entities(poles) in the map will collide with eachother
# IMPORTANT: do this before adding your player 
for pole_a in ENTITIES.values():
    for pole_b in ENTITIES.values():
        if pole_a.id != pole_b.id:
            pole_a.components[ComponentType.Shape_].ignore(pole_b)

```

So the poles are being added in the right locations and drifting to the left. Time to make a bird.

```python
# ComponentType
JumpController_ = 3

# initialize bird above the JumpController
bird = Entity()

# Now to make a modified controller that just responds to the space bar
class JumpController(Component):
    def __init__(self, speed: int = 1, jump_speed: int = 2):
        super().__init__(ComponentType.JumpController_)
        self.speed = speed
        self.jump_speed = jump_speed

    def update(self, game):
        if game.is_key_clicked(pygame.K_SPACE):
            self.entity.position[1] -= self.jump_speed
        if game.is_key_down(pygame.K_a) or game.is_key_down(pygame.K_LEFT):
            self.entity.position[0] -= self.speed
        if game.is_key_down(pygame.K_d) or game.is_key_down(pygame.K_RIGHT):
            self.entity.position[0] += self.speed
        # If bird falls off map remove bird
        if self.entity.position[1] >= DISPLAY_HEIGHT:
            remove_entity(bird)

# in Game.init()
# Gravity
bird.add_component(Drift([0, 1]))
# User controls
bird.add_component(JumpController(1, 75))
bird.add_component(Shape(ShapeType.Circle_, [32], pygame.Color(0xff0000ff)))
add_entity(bird)
```

While this is only a simple game the idea doesn't change as more complexity is added. Instead of copying and pasting sections of code, just make a component and then add it to whatever entity should have that functionality. Another example would be for a game that has many different animals, each of these animals if programmed as seperate object would have almost the same code. It is likely that most programmers would first opt to try an object hierarchy, with animal as the base class, and each type of animal would extend the base class. This method works until you make multiple variants of the same animal, for example a deer would have the same behavior as a moose. In the heirarchal modal, you would need a class for each animal, but in a component model, you only need a component for each behavior. **[Next part](https://threefourseven.github.io/blog/2020/05/27/intro-to-python-game-dev-p4)** will cover making a game with this system.