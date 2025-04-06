---
layout:     post
title:      奥蒙妮游戏实习
subtitle:   2024 暑期实习
date:       2024-07-02
author:     JY
header-img: img/post-bg-cook.jpg
catalog: true
lang: zh
tags:
    - C#
    - Unity
    - Internship
---


### Project Brief

The project is cooperating with Amazon AWS Team, Use reinforcement learning to train AI agent in our gameplay. Our game [Quad Battle](https://store.steampowered.com/app/2525990/Quad_Battle/) is a multiplayer online game, combining MOBA, Monopoly, and auto chess. The game aim to be published by next year, play it if you like!

In this project, I'm mainly responsible includes building and maintaining server, modify communication protocal, provide interface for AWS team, and modify game structure according to requirements.

### RL Design Challenges

Conventional reinforcement learning use game like chess, while this game is much more complex. Player have abilities, and Map have different tools, so many effect were used to define state, action and reward

#### State
1. Player's Position. The map in grid form, so x-y coordinateds can be used to define positon. It's easier then conventional MOBA game, where player's position is continous.

2. Teammates State. The game is usually in a team of 4, so Teammates' state and Enemies' state is also considered. This includes positon, health and score.

3. Map info: there are different tools for player to use on the map, some of then are generated dynamicly, so it's necessary to put this into consideration.

#### Action
This is more complex, as Each character have it's unique ability.

1. Move/Attack:
2. Use ability
3. Buy/Sell minions
4. Make tech choice

#### Reward

1. Winner Score: Once Score reach 3000, the team wins, so this is important
2. castle health: Like most MOBA, if the crystal is destroyed, the team loose immediately, so health of castle including crystal is also important



### AWS Manager Server Design
Quad Battle is a online game, so game needs to run on a server, seperate from AWS training agents. Also, there many be more then one agent in a battle, and more then one battle running at the same time. So it's necessary to build a seperate server, AWS Manager, to Manage diffent battle and agnets.

AWS Manager use HTTP to accept requests.
1. CreateBattle: Create a new battle
2. DeleteeBattle: End a battle
3. UpdateBattle: Update a Battle, make it proceed by a given time interval. 

Since UpdateBattle will be called very frequently, a ProducerConsumerQueue is used to optimize. Update requests will be stored in this messagequeue, and workers can retrive and complete the update task from the queue asynchronously.

For a certain battle, a protobuf protocal is used to define TCP messages between client and server.
Because Training require frequent update of state infomation, I create a messge ReconnectMsg. This is modified from the Message responsible for reconnection, because reconnection require most of the current game status.

### Game Modify
For Reinforcement Learning, a gameplay can be done in under one minute. To improve efficiency, This require us to modify the game, enable it to speed up according to update speed of training agent.


Quad Battle use a ECS Structure, and there's a component corresponding for managing gametime. This component use real-world time to update it's value.


Inorder to achieve game acceleration, I created a new component to handle time. But instead of using real-world time, i set a fixed time for each game frame. So eachtime it's updated, the game is processed by a constant time.

UpdateBattle, the HTTP request handled by AWS Manager, will update the game component. Training agent can send this request to AWS Manager to inform it's ready for it's next step. By doing this, the agent can control the game to go that it's speed.
