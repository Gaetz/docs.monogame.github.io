---
title: "Step 11: Sounds"
description: Improve the game feeling with sounds.
---

# Step 11: Sounds

## Objective

In this step, we will add some sounds to the game. Sounds are a great way to improve the game feeling, as they emphasize the player's actions and world reactions. We will add sounds for the player shooting, for the player and enemy getting hit, when the enemy explodes and when the player gets a power-up.

> [!WARNING]
>
> You are supposed to have read lesson 14 of MonoGame's 2D tutorial. You should know about the `SoundEffect` and `Song` classes. You can improve the code we will write in this lesson by reading chapter 15.

|   Summary                |     Content                                                           |       Link                      |
| ----------------------- | --------------------------------------------------------------------- | ------------------------------- |
| SoundEffects and Music  | Learn how to load and play sound effects and background music         | [2D games chapter 14](https://docs.monogame.net/articles/tutorials/building_2d_games/14_soundeffects_and_music/index.html)          |
| Audio Controller        | A reusable audio controller class to manage sound effects and music   | [2D games chapter 15](https://docs.monogame.net/articles/tutorials/building_2d_games/15_audio_controller/index.html)          |

## Sound for the player shooting

We will present the sound effect system with the player example. Add the sound resources in MGCB.

First, we need to add a `SoundEffect` field to the `Player.cs` class. This field will be used to play the sound when the player shoots. We will load this sound from the content manager in the `Load` method.

```csharp
internal class Player : Entity
{
  ...
  private SoundEffect shootSound;

  ...
  public override void Load(ContentManager content, string modelName)
  {
    base.Load(content, modelName);
    position = new Vector3(0, 0.0f, -250.0f);
    shootSound = content.Load<SoundEffect>("Laser0");
  }

  ...
}
```

> [!IMPORTANT] Requirements
>
> It is important to load the sound in the `Load` method and not just before playing it, because loading a sound takes some time.

We can now play the sound when the player shoots.

```csharp
  public void Update(double dt)
  {
    HandlingInput(dt);
    HandleAiming();

    // Handle shooting
    MouseState mouse = Mouse.GetState();
    if (mouse.LeftButton == ButtonState.Pressed && cooldownTimer <= 0)
    {
      shootSound.Play();
      if (projectileNumber == 1)
      {
        game.AddProjectile(position, orientation, 1000.0f);
      }
      else
      {
        CreateProjectiles();
      }
      cooldownTimer = COOLDOWN;
    }
    cooldownTimer -= (float)dt;

    base.Update(dt);
    boundingBox = CreateBoundingBox();
  }
```

That's it! MonoGame makes sound handling straightforward.

## Sounds in the Game1 class

### Sound fields and loading

As for the `Player`, we need to add some sound fields to the `Game1` class and load them in `LoadContent`.

```csharp
public class Game1 : Game
{
  ...
  private SoundEffect explosionSound;
  private SoundEffect bonusSound;
  private SoundEffect smallExplosionSound;
  private SoundEffect smallExplosionEnemySound;

  ...
  protected override void LoadContent()
  {
    _spriteBatch = new SpriteBatch(GraphicsDevice);

    playerAim = new PlayerAim();
    playerAim.Load(Content, GraphicsDevice);

    player = new Player(playerAim, this);
    player.Load(Content, "Ship");

    levelData = Content.Load<Tutorial_Data.WaveData[]>("Level0");
    waves = LoadWaves(levelData);

    explosionSound = Content.Load<SoundEffect>("Explosion");
    bonusSound = Content.Load<SoundEffect>("PickupBonus");
    smallExplosionSound = Content.Load<SoundEffect>("SmallExplosion");
    smallExplosionEnemySound = Content.Load<SoundEffect>("SmallExplosionEnemy");
  }
  ...
}
```

### Playing power-up sound

This is very simple: call the sound's `Play` function when the player picks up a power-up.

```csharp
  private void UpdatePowerUps(double dt)
  {
    for (int i = powerUps.Count - 1; i >= 0; i--)
    {
      powerUps[i].Update(dt);
      if (player.BoundingBox.Intersects(powerUps[i].BoundingBox))
      {
        player.PowerUp();
        powerUps.RemoveAt(i);
        bonusSound.Play();
        break;
      }
      if (powerUps[i].Position.Z > 500)
      {
        powerUps.RemoveAt(i);
        break;
      }
    }
  }
```

### Playing enemy sounds

First, we remove the explosion particle system addition from `UpdateEnemies`. We will actually add it at the same place where we will play the sound: when the player's projectile hits the enemy.

```csharp
  private void UpdateProjectiles(double dt)
  {
    for (int i = projectiles.Count - 1; i >= 0; i--)
    {
      projectiles[i].Update(dt);
      // Remove projectiles that are out of bounds
      if (projectiles[i].Position.Z < -10000 || projectiles[i].Position.Z > 1000)
      {
        projectiles.RemoveAt(i);
        continue;
      }
      // Collision with player
      if (projectiles[i].BoundingBox.Intersects(player.BoundingBox)
          && !projectiles[i].FromPlayer)
      {
        particleSystems.Add(new ParticleSystem(_graphics.GraphicsDevice, player.Position, 5f, 0.5f, 200f, Color.Orange, Color.Red));
        player.RemoveHp();
        projectiles.RemoveAt(i);
        smallExplosionSound.Play();
        continue;
      }
      // Collision with enemies
      foreach (Enemy enemy in enemies)
      {
        if (enemy.BoundingBox.Intersects(projectiles[i].BoundingBox)
            && projectiles[i].FromPlayer)
        {
          Vector3 enemyPosition = enemy.Position; // Keep enemy position if the enemy is dead
          particleSystems.Add(new ParticleSystem(_graphics.GraphicsDevice, enemyPosition, 5f, 0.5f, 200f, Color.LightGreen, Color.Green));
          enemy.RemoveHp();
          if (enemy.IsDead)
          {
              particleSystems.Add(new ParticleSystem(_graphics.GraphicsDevice, enemyPosition, 10f, 1.5f, 500f, Color.Orange, new Color(100, 0, 0)));
              explosionSound.Play();
          }
          else
          {
              smallExplosionEnemySound.Play();
          }
          projectiles.RemoveAt(i);
          break;
        }
      }
    }
  }
```

As shown, we play `smallExplosion` when the player or enemy is hit (but not dead). If the enemy dies from the hit, we play `explosion` and add the larger particle system.

## Conclusion

In this short step we added sounds to the game and played them in appropriate places. Even small sound additions have a large impact on game feeling.

In the next step, we will use `BasicEffect` further to improve the game's visual feedback.
