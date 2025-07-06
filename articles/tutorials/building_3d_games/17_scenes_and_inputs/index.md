---
title: "Step 17: Scene menu and game over"
description: Create a menu and a gameover scene to wrap up the game.
---

# Step 17: Scenes and inputs

## Objective

In this lesson, we will implement the `SceneMenu.cs` and `SceneGameOver.cs` class. For both those scene, we will need to detect a *Pressed* input, that is an input you press and that was pressed the frame before. We wil create a simple `InputManager.cs` class to do that.

We will then use this input manager to allow our game to be ported to other platform. MomoGame will handle the heavy lifting, but we will still need to indicate which inputs we want to use on each of those platforms.

> [!WARNING]
>
> Input management has already be explained in the basic 2D tutorial. You can refer to it if you want additional information.

|   Sum up                |     Content                                                           |       Link                      |
| ----------------------- | --------------------------------------------------------------------- | ------------------------------- |
| Input management        | How to actually use inputs in a game                                  | [2D games chapter 11](https://docs.monogame.net/articles/tutorials/building_2d_games/11_input_management/index.html)  |

## Preparing the use of other scenes

### Switching to Menu and GameOver in the Game1 class

First, we will prepare the capability to change to our two new scenes in the `Game1.cs` file. We will also create two new public functions for this.

```csharp
public class Game1 : Game
{
    ...
    protected override void LoadContent()
    {
        _spriteBatch = new SpriteBatch(GraphicsDevice);

        SwitchScene(SceneType.Menu);
    }

    ...
    private void SwitchScene(SceneType sceneType)
    {
        switch (sceneType)
        {
            case SceneType.Game:
                currentScene = new SceneGame(this);
                break;
            case SceneType.Menu:
                currentScene = new SceneMenu(this);
                break;
            case SceneType.GameOver:
                currentScene = new SceneGameOver(this);
                break;
        }
        currentScene.Load(Content, GraphicsDevice);
    }

    public void GameOver()
    {
        SwitchScene(SceneType.GameOver);
    }

    public void Start()
    {
        SwitchScene(SceneType.Game);
    }

    public void GoToMenu()
    {
        SwitchScene(SceneType.Menu);
    }
}
```

In order to switch from the menu to the game scene, we need to ba able to detect specific types of key presses. We will handle that by implementing an input manager.

### Managing pressed keys

We can distinguish two ways to press a key: holding it dowm (we will call `IsKeyDown` the function that detect it) and pressing it, which will only detect the first frame when the key is pressed (we will call it `IsKeyPressed`). To make this distinction, we need to keep track of the current key state and the one from the frame before, which will be called `previousKeyboardState`.

Create the `InputManager.cs` file:

```csharp
internal class InputManager
{
    KeyboardState previousKeyboardState;
    KeyboardState currentKeyboardState;
    bool isPreviousStateNull = true;

    public void Update()
    {
        KeyboardState state = Keyboard.GetState();
        currentKeyboardState = state;
    }

    public void EndUpdate()
    {
        previousKeyboardState = currentKeyboardState;
        isPreviousStateNull = false;
    }

    public bool IsKeyPressed(Keys key)
    {
        return currentKeyboardState.IsKeyDown(key) 
            && previousKeyboardState.IsKeyUp(key) 
            && !isPreviousStateNull;
    }

    public bool IsKeyDown(Keys key)
    {
        return currentKeyboardState.IsKeyDown(key);
    }
}
```

We added the `isPreviousStateNull` boolean to manage the first frame of a scene. It may happen that a scene is loaded while a key is being held down - because the player still has its finger on the key after the scene is loaded. We need `IsKeyPressed` to return `false` in this case.

## The game over scene

Our `SceneGameOver.cs` class will be very simple. It will only display an image and wait for the player to press *Enter* or *Space*, to come back to the main menu.

```csharp
internal class SceneGameOver : Scene
{
    private Game1 game;

    Texture2D gameOverPanel;
    Rectangle panelRect = new Rectangle(0, 0, 800, 480);
    InputManager inputManager = new InputManager();

    public SceneGameOver(Game1 game)
    {
        this.game = game;
    }

    public void Load(ContentManager contentManager, GraphicsDevice graphicsDevice)
    {
        gameOverPanel = contentManager.Load<Texture2D>("GameOver");
    }

    public void Update(double dt, GraphicsDevice graphicsDevice)
    {
        inputManager.Update();

        if (inputManager.IsKeyPressed(Keys.Enter) || inputManager.IsKeyPressed(Keys.Space))
        {
            game.GoToMenu();
        }

        inputManager.EndUpdate();
    }

    public void Draw(SpriteBatch spriteBatch, GraphicsDevice graphicsDevice)
    {
        spriteBatch.Begin();
        spriteBatch.Draw(gameOverPanel, panelRect, Color.White);
        spriteBatch.End();
    }
}
```

You can see how we use the input manager. We update it with the key state at the beginning of the frame, and use `EndUpdate` to save the key state for next frame.

## The menu scene

For our `SceneMenu.cs` we will make something beautiful. We will have a 3d scene with a lateral view of the player's ship, moving above the ground. Displayed over it, two menu options: start the game and quit.

The following code will implement that. You will see that we orientated the camera in a different way than in the `SceneGame`. The ship position and orientation will oscillate to give a more dynamic look.

The `menuItem` selector is used to know what to do when a validation key is pressed. It can have value 0 or 1, because we only have two menu options. When the validation is pressed while `menuItem` is 0, the game will be loaded. When `menuItem` is 1, we will quit our game.

```csharp
internal class SceneMenu : Scene
{
    private Game1 game;

    private Matrix view = Matrix.CreateLookAt(new Vector3(-200, 100, -300), new Vector3(100, -100, 0), Vector3.UnitY);
    private Matrix projection = Matrix.CreatePerspectiveFieldOfView(MathHelper.ToRadians(45), 800f / 480f, 1f, 10000f);

    private ShiftingTexture ground;
    private ShiftingTexture sky;
    private Entity ship;
    private Quaternion SHIP_DEFAULT_ORIENTATION = Quaternion.CreateFromAxisAngle(Vector3.Up, MathF.PI);
    double timeCounter = 0;

    Texture2D playButton;
    Texture2D quitButton;
    Texture2D playButtonSelected;
    Texture2D quitButtonSelected;
    int menuItem = 0;

    InputManager inputManager = new InputManager();

    public SceneMenu(Game1 game)
    {
        this.game = game;
    }

    public void Load(ContentManager contentManager, GraphicsDevice graphicsDevice)
    {
        ground = new ShiftingTexture(new Vector3(0, -150, 1000),
            Quaternion.CreateFromAxisAngle(Vector3.Right, -MathF.PI / 2),
            new Vector2(3000f, 3000f),
            new Vector2(0, -0.2f));
        ground.Load(contentManager, graphicsDevice, "Grid");

        sky = new ShiftingTexture(new Vector3(0, 150, 1000),
            Quaternion.CreateFromAxisAngle(Vector3.Right, MathF.PI / 2),
            new Vector2(3000f, 3000f),
            new Vector2(0, 0.2f));
        sky.Load(contentManager, graphicsDevice, "Grid");

        ship = new Entity();
        ship.Load(contentManager, "Ship");
        ship.Position = new Vector3(0, 0, 0);
        ship.Orientation = SHIP_DEFAULT_ORIENTATION;
        ship.Scale = new Vector3(2.0f, 2.0f, 2.0f);

        playButton = contentManager.Load<Texture2D>("PlayButton");
        quitButton = contentManager.Load<Texture2D>("QuitButton");
        playButtonSelected = contentManager.Load<Texture2D>("PlayButtonSelected");
        quitButtonSelected = contentManager.Load<Texture2D>("QuitButtonSelected");
    }

    public void Update(double dt, GraphicsDevice graphicsDevice)
    {
        inputManager.Update();

        ground.Update(dt);
        sky.Update(dt);

        timeCounter += dt;
        float x = (float)Math.Cos(timeCounter);
        ship.Position = new Vector3(x * 50f, 0, 0);
        ship.Orientation = Quaternion.CreateFromAxisAngle(Vector3.Forward, x * -0.2f) * SHIP_DEFAULT_ORIENTATION;
        ship.Update(dt);

        if (inputManager.IsKeyPressed(Keys.S) || inputManager.IsKeyPressed(Keys.Down))
        { 
            menuItem = (menuItem + 1) % 2;
        }
        if (inputManager.IsKeyPressed(Keys.W) || inputManager.IsKeyPressed(Keys.Z) || inputManager.IsKeyPressed(Keys.Up))
        {
            menuItem = (menuItem - 1) % 2;
        }
        if (inputManager.IsKeyPressed(Keys.Enter) || inputManager.IsKeyPressed(Keys.Space))
        {
            if (menuItem == 0)
            {
                game.Start();
            }
            else
            {
                game.Exit();
            }
        }

        inputManager.EndUpdate();
    }

    public void Draw(SpriteBatch spriteBatch, GraphicsDevice graphicsDevice)
    {
        Color bgColor = new Color(30, 0, 50);
        graphicsDevice.Clear(bgColor);

        ground.Draw(view, projection);
        sky.Draw(view, projection);
        ship.Draw(view, projection);

        spriteBatch.Begin();
        if (menuItem == 0)
        {
            spriteBatch.Draw(playButtonSelected, new Vector2(300, 300), Color.White);
            spriteBatch.Draw(quitButton, new Vector2(300, 380), Color.White);
        }
        else
        {
            spriteBatch.Draw(playButton, new Vector2(300, 300), Color.White);
            spriteBatch.Draw(quitButtonSelected, new Vector2(300, 380), Color.White);
        }
        spriteBatch.End();

        graphicsDevice.SamplerStates[0] = SamplerState.LinearWrap;
        graphicsDevice.DepthStencilState = DepthStencilState.Default;
    }
}
```

Because this scene mix 3D and 2D, so we need to reset the graphics device after we use the `SpriteBatch`, as in the `SceneGame`.

## Allowing to port our game to other platforms

Until now, we would directly call the input manager to ask it in which state such key was. This is very straightforward, but will prove difficult if we want our game to work on several platforms with different input types.

The solution is to separate the game actions from the input itself. In our input manager will be asked if such action is pressed, and the action itself will test the input.

We could implement a very general system to manage any kind of action and input, but you have come this far in the tutorial and you have started to understand I do not like unnecessary abstractions. We will implement what just what we need: getting to know if an action is used, and testing the inputs for this action.

### More inputs

First, we now need to support other imput systems. Here, we will support first player gamepad and mouse. We will handle them the same way we have handled the keyboard.

We also change the input detection functions visibility to `private` and add some for button and mouse handling. Note that we will not manage mouse mouvement.

```csharp
internal class InputManager
{
    KeyboardState previousKeyboardState;
    KeyboardState currentKeyboardState;
    GamePadState previousGamePadState;
    GamePadState currentGamePadState;
    MouseState previousMouseState;
    MouseState currentMouseState;

    bool isPreviousStateNull = true;
    bool isPreviousGamePadStateNull = true;
    bool isPreviousMouseStateNull = true;

    public void Update()
    {
        KeyboardState keyboardState = Keyboard.GetState();
        GamePadState gamePadState = GamePad.GetState(PlayerIndex.One);
        MouseState mouseState = Mouse.GetState();
        currentKeyboardState = keyboardState;
        currentGamePadState = gamePadState;
        currentMouseState = mouseState;
    }

    public void EndUpdate()
    {
        previousKeyboardState = currentKeyboardState;
        previousGamePadState = currentGamePadState;
        previousMouseState = currentMouseState;
        isPreviousStateNull = false;
        isPreviousMouseStateNull = false;
        isPreviousGamePadStateNull = false;
    }

    private bool IsKeyPressed(Keys key)
    {
        return currentKeyboardState.IsKeyDown(key) 
            && previousKeyboardState.IsKeyUp(key) 
            && !isPreviousStateNull;
    }

    private bool IsKeyDown(Keys key)
    {
        return currentKeyboardState.IsKeyDown(key);
    }

    private bool IsButtonPressed(Buttons button)
    {
        return currentGamePadState.IsButtonDown(button) 
            && previousGamePadState.IsButtonUp(button) 
            && !isPreviousGamePadStateNull;
    }

    private bool IsButtonDown(Buttons button)
    {
        return currentGamePadState.IsButtonDown(button);
    }

    private bool IsMouseLeftButtonPressed()
    {
        return currentMouseState.LeftButton == ButtonState.Pressed 
        && previousMouseState.LeftButton == ButtonState.Released 
        && !isPreviousMouseStateNull;
    }

    private bool IsMouseLeftButtonDown()
    {
        return currentMouseState.LeftButton == ButtonState.Pressed;
    }
    ...
}
```

### Differentiating actions and inputs

We made the input detection functions `private`. The goal is to no longer use them outside our `InputManager` class.

Instead, we will create `public` "action" function that will correspond to the different input we want to be associated with one action. For instance, going up will be associated with holding the W, Z or Up keys, or maintaining the gamepad's left thumb stick, or its up directional pad button.

```csharp
internal class InputManager
{
    ...
    public bool IsValidationActionPressed()
    {
        return IsKeyPressed(Keys.Enter) || IsKeyPressed(Keys.Space) 
            || IsButtonPressed(Buttons.A) || IsButtonPressed(Buttons.RightShoulder) 
            || IsButtonPressed(Buttons.RightTrigger);
    }

    public bool IsShootActionPressed()
    {
        return IsMouseLeftButtonPressed() || IsButtonPressed(Buttons.RightTrigger)
            || IsButtonPressed(Buttons.RightShoulder) || IsButtonPressed(Buttons.A)
            || IsKeyPressed(Keys.Space) || IsKeyPressed(Keys.Enter);
    }

    public bool IsUpActionDown() 
    {
        return IsKeyDown(Keys.W) || IsKeyDown(Keys.Up) || IsKeyDown(Keys.Z)
            || IsButtonDown(Buttons.DPadUp) || IsButtonDown(Buttons.LeftThumbstickUp);
    }

    public bool IsDownActionDown() 
    {
        return IsKeyDown(Keys.S) || IsKeyDown(Keys.Down) 
            || IsButtonDown(Buttons.DPadDown) || IsButtonDown(Buttons.LeftThumbstickDown);
    }

    public bool IsLeftActionDown() 
    {
        return IsKeyDown(Keys.A) || IsKeyDown(Keys.Left) || IsKeyDown(Keys.Q)
            || IsButtonDown(Buttons.DPadLeft) || IsButtonDown(Buttons.LeftThumbstickLeft);
    }

    public bool IsRightActionDown() 
    {
        return IsKeyDown(Keys.D) || IsKeyDown(Keys.Right) 
            || IsButtonDown(Buttons.DPadRight) || IsButtonDown(Buttons.LeftThumbstickRight);
    }

    public bool IsUpActionPressed()
    {
        return IsKeyPressed(Keys.W) || IsKeyPressed(Keys.Up) || IsKeyPressed(Keys.Z)
            || IsButtonPressed(Buttons.DPadUp) || IsButtonPressed(Buttons.LeftThumbstickUp);
    }

    public bool IsDownActionPressed()
    {
        return IsKeyPressed(Keys.S) || IsKeyPressed(Keys.Down)
            || IsButtonPressed(Buttons.DPadDown) || IsButtonPressed(Buttons.LeftThumbstickDown);
    }

    public bool IsLeftActionPressed()
    {
        return IsKeyPressed(Keys.A) || IsKeyPressed(Keys.Left) || IsKeyPressed(Keys.Q)
            || IsButtonPressed(Buttons.DPadLeft) || IsButtonPressed(Buttons.LeftThumbstickLeft);
    }

    public bool IsRightActionPressed()
    {
        return IsKeyPressed(Keys.D) || IsKeyPressed(Keys.Right)
            || IsButtonPressed(Buttons.DPadRight) || IsButtonPressed(Buttons.LeftThumbstickRight);
    }
}
```

We have thus defined actions for:

- Menu validation
- Shoot action
- Up, down, left and right directions held down for the ship maneuvers
- Up and down buttons pressed for menus
- Left and right buttons pressed are not used but it does not hurt to define them

Each action can be triggered by the keyboard, the mouse or the gamepad.

### Updating menu and game over scenes

Because we changed the input visibility functions, we now have to use our new action functions in the `SceneMenu` and `SceneGameOver`. The changes are easy to understand.

For the `SceneMenu`:

```csharp
internal class SceneMenu : Scene
{
    ...
    public void Update(double dt, GraphicsDevice graphicsDevice)
    {
        inputManager.Update();

        ground.Update(dt);
        sky.Update(dt);

        timeCounter += dt;
        float x = (float)Math.Cos(timeCounter);
        ship.Position = new Vector3(x * 50f, 0, 0);
        ship.Orientation = Quaternion.CreateFromAxisAngle(Vector3.Forward, x * -0.2f) * SHIP_DEFAULT_ORIENTATION;
        ship.Update(dt);

        if (inputManager.IsDownActionPressed())
        { 
            menuItem = (menuItem + 1) % 2;
        }
        if (inputManager.IsUpActionPressed())
        {
            menuItem = (menuItem - 1) % 2;
        }
        if (inputManager.IsValidationActionPressed())
        {
            if (menuItem == 0)
            {
                game.Start();
            }
            else
            {
                game.Exit();
            }
        }

        inputManager.EndUpdate();
    }
    ...
}
```

We just call `InputManager.IsDownActionPressed`, `InputManager.IsUpActionPressed` and `InputManager.IsValidationActionPressed` instead of the previous input functions.

For the `SceneGameOver`:

```csharp
internal class SceneGameOver : Scene
{
    ...
    public void Update(double dt, GraphicsDevice graphicsDevice)
    {
        inputManager.Update();

        if (inputManager.IsValidationActionPressed())
        {
            game.GoToMenu();
        }

        inputManager.EndUpdate();
    }
    ...
}
```

### Updating the player

Finally, in the `SceneGame`, inputs are managed by the player - if we exemple the `PlayerAim` move that we will not change here. The `Player` class needs to be updated:

```csharp
internal class Player : Entity
{
    ...
    private void HandlingInput(double dt)
    {
        if (inputManager.IsUpActionDown())
        {
            speedY += ACCELERATION_RATE * (float)dt;
        }
        if (inputManager.IsDownActionDown())
        {
            speedY -= ACCELERATION_RATE * (float)dt;
        }
        if (MathF.Abs(speedY) > MAX_SPEED)
        {
            speedY = MathF.Sign(speedY) * MAX_SPEED;
        }


        if (inputManager.IsLeftActionDown())
        {
            speedX -= ACCELERATION_RATE * (float)dt;
        }
        if (inputManager.IsRightActionDown())
        {
            speedX += ACCELERATION_RATE * (float)dt;
        }
        if (MathF.Abs(speedX) > MAX_SPEED)
        {
            speedX = MathF.Sign(speedX) * MAX_SPEED;
        }


        position += new Vector3((float)(speedX * dt), (float)(speedY * dt), 0);
        if (position.X < BOUNDS.Left)
        {
            position.X = BOUNDS.Left;
            speedX = 0;
        } 
        else if (position.X > BOUNDS.Right)
        {
            position.X = BOUNDS.Right;
            speedX = 0;
        }
        if (position.Y < BOUNDS.Top)
        {
            position.Y = BOUNDS.Top;
            speedY = 0;
        }
        else if (position.Y > BOUNDS.Bottom)
        {
            position.Y = BOUNDS.Bottom;
            speedY = 0;
        }

        speedX *= DECELERATION_RATE;
        speedY *= DECELERATION_RATE;
    }
    ...

    public override void Update(double dt)
    {
        inputManager.Update();

        HandlingInput(dt);
        HandleAiming();

        // Handle shooting
        if (inputManager.IsShootActionPressed() && cooldownTimer <= 0)
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
        laserAim.Update(dt);

        inputManager.EndUpdate();
    }
    ...
}
```

That's it! If you test the game... it should not be different. Except this time you can use a gamepad or easily update your input bindings, thanks to the action functions.

## Conclusion

With the two previous steps, we implemented scene management and our new `SceneMenu` and `SceneGameOver`, and allow the player to control the game with different kinds of input.

In the next and last lesson, we will wrap up everything and add the little small details that will make our demo game shine!
