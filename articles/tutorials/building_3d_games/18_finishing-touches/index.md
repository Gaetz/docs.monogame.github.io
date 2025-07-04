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