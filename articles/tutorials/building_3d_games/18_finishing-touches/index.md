---
title: "Step 18: Finishing touches"
description: Setup the music, continue level and fix player's collisions
---

# Step 18: Finishing touches and conclusions

## Objective

This last lesson is a conclusion. We will grant to our game prototype the last touches it deserves and conclude on your future journey as a MonoGame game developer!

## Polish is wonderful

In video game development, once we have finished the first 90% of a game, we tend to say that we just have to do the "second 90%". That refers to the polish phase, when the developers spend their time adjusting every moving piece of the game to reach a level of feeling and finition that will give the player this mysterious sense of quality and depth. Let's be real: polish is the only way to make your game good - given its core is already fun.

In this tutorial, we will not go through a complete polish phase. It would be too long. We will juste change some elements of the game to give you the direction you would have to follow, if you would like to finish the game we started.

### Music

First, we will improve the genersal mood of the game. The best tool for this is music.

Add the two mp3 files from the resources in the MGCB and build them. We will use one music for the menu scene, one in the game scene, and we will make sure the game over screen stops the music.

Here is for `SceneMenu`:

```csharp
internal class SceneMenu : Scene
{
    ...
    private Song titleSong;

    public void Load(ContentManager contentManager, GraphicsDevice graphicsDevice)
    {
        ...
        titleSong = contentManager.Load<Song>("CatNoisettes-BertrandToupet");
        MediaPlayer.Play(titleSong);
    }
    ...
}
```

As you can see, music songs are loaded from the content manager, then we use `MediaPlayer` to play the music - contrary to a `SoundEffect` that plays itself.

Now for `SceneGame`:

```csharp
internal class SceneGame : Scene
{
    ...
    private Song gameSong;

    public void Load(ContentManager contentManager, GraphicsDevice graphicsDevice)
    {
        ...
        gameSong = contentManager.Load<Song>("SpaceAttack-BertrandToupet");
        MediaPlayer.Play(gameSong);
    }
    ...
}
```

And finally, for `SceneGameOver` we will stop any music playing:

```csharp
internal class SceneGameOver : Scene
{
    ...
    public void Load(ContentManager contentManager, GraphicsDevice graphicsDevice)
    {
        gameOverPanel = contentManager.Load<Texture2D>("GameOver");
        if (MediaPlayer.State == MediaState.Playing)
        {
            MediaPlayer.Stop();
        }
    }
    ...
}
```

That's it! If you start the game, your 3D shooter experience will be much more intense thanks to the power of music!

> [!TIP]
>
> Except if you need to test an ambiance, it is often better to start the game musics at the end of development. Continuously hearing the same music along game development would drive you crazy!

### Player's collision

Currently, our collision boxes are way too big. This is great to test game interactions, but a player would be frustrated to be hit by an enemy's projectile while the graphics show a situation that should have led to a dodge. So we will reduce the size of the `Player` collision box:

```csharp
internal class Player : Entity
{
    ...
    private BoundingBox CreateBoundingBox()
    {
        Vector3[] vertices = {
            new Vector3(-60f, -7f, -67f), new Vector3(60f, -7f, -67f),
            new Vector3(-60f, 25f, -67f), new Vector3(60f, 25f, -67f),
            new Vector3(-60f, -7f, 45f), new Vector3(60f, -7f, 45f),
            new Vector3(-60f, 25f, 45f), new Vector3(60f, 25f, 45f)
        };
        for (int i = 0; i < vertices.Length; i++)
        {
            vertices[i] = Vector3.Transform(vertices[i], world);
        }
        return BoundingBox.CreateFromPoints(vertices);
    }
    ...
}
```

In the box's vertices creation, we have change the *x* value from *64f* to *60f*, and more importantly, positive *z* value from *77f* to *45f*. This will make the player's collision more accurate.

You can work on collisions for all your game objects. Collisions, when done right, will make your game feels more physical, more grounded in reality.

### Expanding the level

Until now, we have two waves of enemies, separated by a power up. You can create other waves with other enemy patterns, main phase timing. The waves system even allows to have several waves present on screen at the same time, provided you time the wave with close enough timings. You can also add commentaries from the robot or from our dauntless pilot. 

You can also imagine to create other kind of enemies, with different patterns - that is to say, a different internal state machine. A level boss, with infinite (-1) main phase duration, could be such an enemy.

Here are two additional waves in the `Level0.xml` file:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<XnaContent>
    <Asset Type="Tutorial_Data.WaveData[]">
        <Item>
            <id>0</id>
            <time>12</time>
            <elementNumber>3</elementNumber>

            <element0Type>enemy</element0Type>
            <element0EnterSide>Left</element0EnterSide>
            <element0ExitSide>Right</element0ExitSide>
            <element0X>-200</element0X>
            <element0Y>0</element0Y>
            <element0Z>-750</element0Z>
            <element0Duration>5.0</element0Duration>

            <element1Type>enemy</element1Type>
            <element1EnterSide>Left</element1EnterSide>
            <element1ExitSide>Right</element1ExitSide>
            <element1X>200</element1X>
            <element1Y>0</element1Y>
            <element1Z>-750</element1Z>
            <element1Duration>5.0</element1Duration>

            <element2Type>enemy</element2Type>
            <element2EnterSide>Left</element2EnterSide>
            <element2ExitSide>Right</element2ExitSide>
            <element2X>0</element2X>
            <element2Y>0</element2Y>
            <element2Z>-750</element2Z>
            <element2Duration>5.0</element2Duration>

            <element3Type>none</element3Type>
            <element3EnterSide>Left</element3EnterSide>
            <element3ExitSide>Right</element3ExitSide>
            <element3X>0</element3X>
            <element3Y>0</element3Y>
            <element3Z>0</element3Z>
            <element3Duration>5.0</element3Duration>

            <element4Type>none</element4Type>
            <element4EnterSide>Left</element4EnterSide>
            <element4ExitSide>Right</element4ExitSide>
            <element4X>0</element4X>
            <element4Y>0</element4Y>
            <element4Z>0</element4Z>
            <element4Duration>5.0</element4Duration>
        </Item>
        
        <Item>
            <id>1</id>
            <time>24</time>
            <elementNumber>1</elementNumber>

            <element0Type>powerup</element0Type>
            <element0EnterSide>Left</element0EnterSide>
            <element0ExitSide>Right</element0ExitSide>
            <element0X>0</element0X>
            <element0Y>0</element0Y>
            <element0Z>-500</element0Z>
            <element0Duration>5.0</element0Duration>

            <element1Type>none</element1Type>
            <element1EnterSide>Left</element1EnterSide>
            <element1ExitSide>Right</element1ExitSide>
            <element1X>200</element1X>
            <element1Y>0</element1Y>
            <element1Z>-750</element1Z>
            <element1Duration>5.0</element1Duration>

            <element2Type>none</element2Type>
            <element2EnterSide>Left</element2EnterSide>
            <element2ExitSide>Right</element2ExitSide>
            <element2X>0</element2X>
            <element2Y>0</element2Y>
            <element2Z>-750</element2Z>
            <element2Duration>5.0</element2Duration>

            <element3Type>none</element3Type>
            <element3EnterSide>Left</element3EnterSide>
            <element3ExitSide>Right</element3ExitSide>
            <element3X>0</element3X>
            <element3Y>0</element3Y>
            <element3Z>0</element3Z>
            <element3Duration>5.0</element3Duration>

            <element4Type>none</element4Type>
            <element4EnterSide>Left</element4EnterSide>
            <element4ExitSide>Right</element4ExitSide>
            <element4X>0</element4X>
            <element4Y>0</element4Y>
            <element4Z>0</element4Z>
            <element4Duration>5.0</element4Duration>
        </Item>
        
        <Item>
            <id>2</id>
            <time>27</time>
            <elementNumber>3</elementNumber>

            <element0Type>enemy</element0Type>
            <element0EnterSide>Bottom</element0EnterSide>
            <element0ExitSide>Bottom</element0ExitSide>
            <element0X>-200</element0X>
            <element0Y>0</element0Y>
            <element0Z>-750</element0Z>
            <element0Duration>5.0</element0Duration>

            <element1Type>enemy</element1Type>
            <element1EnterSide>Bottom</element1EnterSide>
            <element1ExitSide>Bottom</element1ExitSide>
            <element1X>200</element1X>
            <element1Y>0</element1Y>
            <element1Z>-750</element1Z>
            <element1Duration>5.0</element1Duration>

            <element2Type>enemy</element2Type>
            <element2EnterSide>Bottom</element2EnterSide>
            <element2ExitSide>Bottom</element2ExitSide>
            <element2X>0</element2X>
            <element2Y>0</element2Y>
            <element2Z>-750</element2Z>
            <element2Duration>5.0</element2Duration>

            <element3Type>none</element3Type>
            <element3EnterSide>Bottom</element3EnterSide>
            <element3ExitSide>Bottom</element3ExitSide>
            <element3X>0</element3X>
            <element3Y>0</element3Y>
            <element3Z>0</element3Z>
            <element3Duration>5.0</element3Duration>

            <element4Type>none</element4Type>
            <element4EnterSide>Bottom</element4EnterSide>
            <element4ExitSide>Bottom</element4ExitSide>
            <element4X>0</element4X>
            <element4Y>0</element4Y>
            <element4Z>0</element4Z>
            <element4Duration>5.0</element4Duration>
        </Item>

        <Item>
            <id>3</id>
            <time>37</time>
            <elementNumber>5</elementNumber>

            <element0Type>enemy</element0Type>
            <element0EnterSide>Top</element0EnterSide>
            <element0ExitSide>Bottom</element0ExitSide>
            <element0X>-200</element0X>
            <element0Y>50</element0Y>
            <element0Z>-750</element0Z>
            <element0Duration>5.0</element0Duration>

            <element1Type>enemy</element1Type>
            <element1EnterSide>Top</element1EnterSide>
            <element1ExitSide>Bottom</element1ExitSide>
            <element1X>200</element1X>
            <element1Y>-50</element1Y>
            <element1Z>-750</element1Z>
            <element1Duration>5.0</element1Duration>

            <element2Type>enemy</element2Type>
            <element2EnterSide>Top</element2EnterSide>
            <element2ExitSide>Bottom</element2ExitSide>
            <element2X>0</element2X>
            <element2Y>0</element2Y>
            <element2Z>-750</element2Z>
            <element2Duration>5.0</element2Duration>

            <element3Type>enemy</element3Type>
            <element3EnterSide>Top</element3EnterSide>
            <element3ExitSide>Bottom</element3ExitSide>
            <element3X>-100</element3X>
            <element3Y>25</element3Y>
            <element3Z>-750</element3Z>
            <element3Duration>5.0</element3Duration>

            <element4Type>enemy</element4Type>
            <element4EnterSide>Top</element4EnterSide>
            <element4ExitSide>Bottom</element4ExitSide>
            <element4X>100</element4X>
            <element4Y>-25</element4Y>
            <element4Z>-750</element4Z>
            <element4Duration>5.0</element4Duration>
        </Item>

        <Item>
            <id>4</id>
            <time>47</time>
            <elementNumber>5</elementNumber>

            <element0Type>enemy</element0Type>
            <element0EnterSide>Top</element0EnterSide>
            <element0ExitSide>Bottom</element0ExitSide>
            <element0X>-200</element0X>
            <element0Y>-50</element0Y>
            <element0Z>-750</element0Z>
            <element0Duration>5.0</element0Duration>

            <element1Type>enemy</element1Type>
            <element1EnterSide>Top</element1EnterSide>
            <element1ExitSide>Bottom</element1ExitSide>
            <element1X>200</element1X>
            <element1Y>50</element1Y>
            <element1Z>-750</element1Z>
            <element1Duration>5.0</element1Duration>

            <element2Type>enemy</element2Type>
            <element2EnterSide>Top</element2EnterSide>
            <element2ExitSide>Bottom</element2ExitSide>
            <element2X>0</element2X>
            <element2Y>0</element2Y>
            <element2Z>-750</element2Z>
            <element2Duration>5.0</element2Duration>

            <element3Type>enemy</element3Type>
            <element3EnterSide>Top</element3EnterSide>
            <element3ExitSide>Bottom</element3ExitSide>
            <element3X>-100</element3X>
            <element3Y>-25</element3Y>
            <element3Z>-750</element3 Z>
            <element3Duration>5.0</element3Duration>

            <element4Type>enemy</element4Type>
            <element4EnterSide>Top</element4EnterSide>
            <element4ExitSide>Bottom</element4ExitSide>
            <element4X>100</element4X>
            <element4Y>25</element4Y>
            <element4Z>-750</element4Z>
            <element4Duration>5.0</element4Duration>
        </Item>
    </Asset>
</XnaContent>
```

### Other ideas

This is just a beginning. In this tutorial, we just set up the main mecanic of our game. It's your turn to think about the rules and variations you want to give to your player, to make sure they have a wonderful experience playing your game. Now you know how to make 3D games: a new dimension of creation literaly just opened up.

## Packaging and publishing your game

MonoGame allows you to create a game executable, that you can share with your players or sell on an online store. The details on how to do so were covered in the [Packaging Your Game for Distribution](https://docs.monogame.net/articles/tutorials/building_2d_games/25_packaging_game/index.html) article, so I will forward you to it :)

If you want to share your game with the world, you could publish your game on the [Itch.io](https://itch.io/) platform, which is a great place to share your indie games. The [Publishing your game on Itch.io](https://docs.monogame.net/articles/tutorials/building_2d_games/26_publishing_game/index.html) article will help you to do so.

## Conclusion and next steps

Congratulations! You have reach the end of the MonoGame's basic 3D tutorial. I hope you learned useful things and realize how a convenient framework MonoGame is - And I mean it, for I have tested dozens of frameworks and engines. If you have a dream 3D game project, well, go for it. You will learn while making it. If you are not adamant on the kind of game you want to create, I would advise you to create some small and simple gameplay prototypes, so you get use to the 3D thinking. Chances are one of your prototypes will feel funnier than the other to make, and you will create a game out of it!

As for myself, I am glad to have accompany you until here. I did my best to create a tutorial that would be at the same time lightweight and informative. Of course, it is far from perfect. If you have suggestions about how to improve it, feel free to post an issue on the monogame documentation github.

Farewell, I wish you a pleasant video game creation journey. See you on the way!
