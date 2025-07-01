---
title: "Step 17: Scene menu and game over"
description: Create a menu and a gameover scene to wrap up the game.
---

# Step 17: Scenes and inputs

## Objective

In this lesson, we will implement the `SceneMenu.cs` and `SceneGameOver.cs` class. For both those scene, we will need to detect a *Pressed* input, that is an input you press and that was pressed the frame before. We wil create a simple `InputManager.cs` class to do that.

We will then use this input manager to allow our game to be ported to other platform. MomoGame will handle the heavy lifting, but we will still need to indicate which inputs we want to use on each of those platforms.

## Requierements

Inpit management has already be explained in the basic 2D tutorial. You can refer to it if you want additional information.

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

We can distinguish two ways to press a key: holding it dowm (we will call `IsKeyDown` the function that detect it) and pressing it, which will only detect the first frame when the key is pressed (we will call it `IsKeyPressed`). To make this distinction, we need to keep track of the current key state and the one from the frame before, which will be called `previousState`.

Create the `InputManager.cs` file:

```csharp
internal class InputManager
{
    KeyboardState previousState;
    KeyboardState currentState;
    bool isPreviousStateNull = true;

    public void Update(KeyboardState state)
    {
        currentState = state;
    }

    public void EndUpdate()
    {
        previousState = currentState;
        isPreviousStateNull = false;
    }

    public bool IsKeyPressed(Keys key)
    {
        return currentState.IsKeyDown(key) 
            && previousState.IsKeyUp(key) 
            && !isPreviousStateNull;
    }

    public bool IsKeyDown(Keys key)
    {
        return currentState.IsKeyDown(key);
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
        KeyboardState state = Keyboard.GetState();
        inputManager.Update(state);
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
        KeyboardState state = Keyboard.GetState();
        inputManager.Update(state);

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

### A more general input manager






## Conclusion

With the two previous steps, we implemented scene management and our new `SceneMenu` and `SceneGameOver`.

In the next and last lesson, we will wrap up everything and add the little small details that will make our demo game shine!