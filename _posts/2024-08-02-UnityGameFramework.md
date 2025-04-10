---
title:      Game Framework ECS Introduction
header:
  overlay_image: "assets/wide_bgs/park.jpg"
tags:
    - Unity
    - C#
---

## Preface

Recently, during my internship, I encountered a lot of game-related architecture and knowledge, including the ECS architecture. However, the information is vast and somewhat scattered, so I decided to organize it.

## Main Content

### Introduction to ECS
In large projects, the most commonly used architecture is the ECS (Entity Component System) architecture.

ECS, as the name suggests, is a structure composed of Entity, Component, and System. Of course, the actual situation is much more complex, but the core remains these three concepts:

#### Entity:

- A game object or concept
- Contains an ID, a unique identifier
- Does not have its own logic or data

#### Component:

- Pure data, no logic
- Represents a feature of an entity

#### **System:**

- Handles the logic, a system typically processes specific components
- A system is usually driven by the game loop to modify the state of game objects
- Iterates over entities and updates their states
<!-- - The `OnProcess` or `ProcessList` in the System will be called in the `game loop` instead of being directly called -->

For example, an entity can be a player in the game, components would be various attributes of the player, such as health, stamina, and mana. The system processes the components, with the most common system being the physics system in a game, which needs to periodically check and update the component representing the player's position.

### Why Use ECS?
Why use ECS instead of traditional OOP for game development? (Overwatch is an example that uses ECS) In short, ECS has performance and language simplicity advantages in game development:

#### Easy Management
A simple example: the game needs several enemies: tanks, infantry, and ghosts. If we use OOP, these three enemies would inherit from the Enemy class. However, ghosts do not have collision volumes, which conflicts with the collision volume included in the Enemy class. But if we use ECS, we just need to not give the ghost entity the collision volume component, and the problem is solved.

Of course, in this situation, we could also solve this by setting the ghost's collision volume to 0. This is even simpler, and OOP does perform well in small projects. But consider this scenario: the game needs to add a new enemy: ghost infantry. It is similar to regular infantry, but without a collision volume. In OOP, this entity can only inherit from the infantry class since it shares many similarities with the infantry. As for the ghost logic? It can only be copied and pasted from the ghost class. Not to mention that as the project grows larger, with hundreds of entities and each entity having dozens of components, it becomes more complex.

<!-- In ECS, you just need to add the components of the infantry (shooting, rendering) and the ghost (collision volume) to the entity. -->

#### Performance Advantages

*Increase CPU cache hits and improve performance*

Entities and systems do not store data, meaning data is entirely managed by components. The data of the same type of component is stored in adjacent locations on the hard drive (even though it may belong to different entities), making CPU access very efficient. This is due to the CPU's caching mechanism, where each time data is read from the hard drive, not only the specific address is read, but data from adjacent locations is also read.

For example, the physics system needs to update the positions of all entities in the game:


```python
# OOP:
for entity in entities:
    if entity.Physics != None:
        entity.Physics.Update()
```

```python
# ECS
for component in physicsComponents:
    component.Update()
```
Obviously, ECS avoids iterating over all entities. OOP not only iterates over unnecessary entities but also accesses data scattered across different memory locations, with the back-and-forth jumping of addresses consuming a lot of time.

At a deeper level, ECS uses memory alignment and entity memory allocators, and Chunk design to optimize memory management, which won't be elaborated on here.

*Decoupling Data and Logic*

As mentioned above, components are responsible for data, while systems are responsible for logic, achieving the decoupling of data and logic. This makes it easier for multi-core CPUs to handle parallel processing, allowing different systems to process different component data concurrently.


### How ECS Runs a Game

A game is generally driven by a main loop, which calls the Update function to update the game each frame. Game updates, essentially, are system updates, as they contain all the logic, and calling them to modify data, which is the component, is sufficient.

#### Classification of Systems
Systems are generally divided into two types, React System and Update System. Regardless of the type of system, there are two necessary functions: Process() and OnProcess().

- OnProcess receives an Entity object and updates a specific component of the Entity.
- Process updates the entire system and calls OnProcess for all entities that need updating.

UpdateSystem represents systems that need to be updated regularly, such as health recovery or the physics system. These systems generally update all entities with specific components, so finding all such entities and calling OnProcess is enough.

ReactSystem represents systems triggered by specific conditions, so which entities need updating is determined by other factors. This type of system stores a _listCache to store the entities that need updating.

### Client and Server
Ideally, assuming the server has infinite performance, all game logic would be calculated by the server, and the server would only need to transmit data to the client for display. This reduces the burden on the client, no longer requiring high-performance CPUs and GPUs, and prevents any cheating since hacking the server would be overkill.

But usually, the client will handle part of the game logic. If the server handles all the computations, the processing load would be overwhelming, especially for large online games like LOL, which may have tens of thousands of games running simultaneously, leading to latency issues. This also creates opportunities for cheating. In the most common FPS games, shooting calculations are typically handled by the client. If they were calculated by the server and then sent back to the client, it would cause severe delays, such as seeing yourself shoot, but the bullet isn't fired until half a second later. Cheats can modify the client's data, such as bullet positions, to cheat, commonly known as magic bullets.

Of course, the game can regularly check server data or use anti-cheat plugins to detect modifications. But these methods only increase the cost of cheating and cannot completely prevent it. After all, the machine is in the player's hands, giving them absolute control. No matter how powerful anti-cheat software is, it cannot access the underlying memory, but the player can. This is the principle behind DMA (Direct Memory Access) cheats, which cheat by directly accessing and modifying memory.

#### How to Communicate?

A game usually have a CoreGame, which is the core logic of the game. This logic is constructed using the ECS architecture. TBA

---
### 参考
- https://blog.csdn.net/u012861978/article/details/132397770
