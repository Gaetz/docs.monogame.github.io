---
title: "Introduction: A plan for our 3D game"
description: Some thought before creating our game and know where to find required monogame knowledge.
---

# The 3D game we will create

Hi! Welcome to this monogame 3D beginner tutorial. Our goal will be to create a fun and effective game that will present the basics of the Monogame API for 3D games. In this extent, we will develop the prototype of a Star Fox 64 game.

Star fox 64 is a 3D shooter game, born at a time where 3D games were still figuring out their controls and their gameplay shapes. It was reknown for its fun controls, its fast paced rythm and its barrel rolls! It basic gameplay is simple enough to be implemented by a beginner in 3D programming, yet you will be able to polish it a lot if you want to make a complete game out of this tutorial. Though we won't implement the barrel roll feature, we will go through all the mathematical and technical elements you need to bootstrap your 3D gameplays, thanks to the Monogame API.

In order to differ from Star Fox, we will choose to make our game taking place in the internal core of a fantasy computer, where our pilot heroin will rush her miniaturized virtual space ship to shoot and destroy nasty bugs! Our final result will look like this:
[gameplay video or image]

Before we start coding, let's review what you need to know.

## Knowledge requirements

You are supposed to be able to create a Monogame 2D game. Thus, I will suppose you have read the wonderul Aristurtle Monogame 2D tutorial. More precisely:

|   Sum up                |     Content                                                           |       Link                      |
| ----------------------- | --------------------------------------------------------------------- | ------------------------------- |
| What is Monogame?       | An high level view and a summary of Monogame's history and features   | Link to Aristurtle's lesson 01  |
| How to install Monogame | Setting up and getting started with Monogame, whichever your platform | Link to Aristurtle's lesson 02  |
| The Game1 class         | Understanding the game loop in the Game1 class                        | Link to Aristurtle's lesson 03  |
| The Content pipeline    | How to load assets using the MGCB editor and the ContentManager class | Link to Aristurtle's lesson 06  |
| Working with textures   | We will need to display textures in our game, for UI.                 | Link to Aristurtle's lesson 07  |
| Handling input          | They way to use keyboard, mouse and controller in Monogame            | Link to Aristurtle's lesson 11  |
| Input management        | How to actually use the previous lesson in a game                     | Link to Aristurtle's lesson 12  |
| Sounds in Monogame      | How to use sound effects and music in your game                       | Link to Aristurtle's lesson 14  |

By the way, you are supposed to be able to use the C# language, for this is the language Monogame is written with. If it is not the case, I would recommend reading the [Learn C#](https://dotnet.microsoft.com/en-us/learn/csharp) tutorials from Microsoft.

The rest of the requiered knowledge to create a 3D game, and specifically 3D mathematics, will be presented in this tutorial.

Now, let's see how we will create our 3D Star Fox clone.

## Our way to a 3D game

This tutorial is divided in 15 steps, which all will tackle a gameplay feature through its Monogame implementation. The steps are the following:

|   Sum up                        |     Content                                                               |       Link           |
| ------------------------------- | ------------------------------------------------------------------------- | -------------------- |
| Step 1: Display a 3D ship       | Load and display the 3D model of a space ship, using the ContentManager   | Link to lesson 01    |
| Step 2: Moving the ship         | Using Vector3 and inputs to move the ship, with 3D velocity               | Link to lesson 02    |
| Step 3: Aim and rotate          | Display a Quad and use rotations with Matrices and Quaternions            | Link to lesson 03    |
| Step 4: Shooting projectiles    | Spawn moving projectiles and use Inheritance for the game's entities      | Link to lesson 04    |
| Step 5: Projectiles' collisions | Handling projectiles collisions on a basic Enemy class                    | Link to lesson 05    |
| Step 6: A power up              | Colliding the player with a power up will improve it's shooting power     | Link to lesson 06    |
| Step 7: A moving enemy          | Make the enemy a State Machine to organize its entry and exit             | Link to lesson 07    |
| Step 8: A shooting enemy        | Extend the enemy's State machine to make it shoot at the player           | Link to lesson 08    |
| Step 9: Waves of enemy          | Using a data library and XML parsing to setup timed waves of enemies      | Link to lesson 09    |
| Step 10: Particles              | Reuse our Quad class to spawn particules, thanks to a Particle system     | Link to lesson 10    |
| Step 11: Sounds and music       | Make our game more impactful adding sounds and a background music         | Link to lesson 11    |
| Step 12: Graphics polishing     | Use further the BasicEffect to make the game feel for reactive            | Link to lesson 12    |
| Step 13: Conversation and 2D UI | Addi a 2D UI on top of our 3D game in order to enable storytelling        | Link to lesson 13    |
| Step 14: A moving floor         | Enhance the game's graphics with a moving floor, to give a speed feeling  | Link to lesson 14    |
| Step 15: Going further          | Plenty of gameplay ideas to extend this game tutorial beyond!             | Link to lesson 15    |

In case you need a reference, you can find the complete code for this tutorial om the following git reposotory: https://github.com/Gaetz/monogame-tutorial-3d

The solution is organized with one project by step. In each project, the classes are duplicated, so you can find the state of each class at this specific step.

## The game's assets

All assets for the game are provided in a resource zipped folder. Make sure to unzip this folder somewhere on your computer before you start coding the project.

# About this documentation

## Conventions Used in This Documentation

The following conventions are used in this documentation

### Italics

*Italics* are used for emphasis, technical terms, and paths such as file paths including filenames and extensions.

### Inline Code Blocks

`Inline code` blocks are used for methods, functions, and variable names when they are discussed with the body a text paragraph.

### Code Blocks

```cs
// Example Code Block
public void Foo() { }
```

Code blocks are used to show code examples with syntax highlighting

## MonoGame

If you ever have questions about MonoGame or would like to talk with other developers to share ideas or just hang out with us, you can find us in the various MonoGame communities below

* [Discord](https://discord.gg/monogame)
* [GitHub Discussions Forum](https://github.com/MonoGame/MonoGame/discussions)
* [Community Forums (deprecated)](https://community.monogame.net/)
* [Reddit](https://www.reddit.com/r/monogame/)
* [Facebook](https://www.facebook.com/monogamecommunity)

# Note From Author

> Monogame (and its predecessor XNA) has been my first game framework, back in 2010, and if we except the time I coded in Basic on my high school hand calculator TI-83.
>
> Since then, blablablou
>
> With this documentation, blablobli
>
> \- Gaëtan Blaise-Cazalet (Gaetz)
