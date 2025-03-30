---
title: "Step 15: Message system"
description: Replace the TargetAim by a cube that imitate the laser aim.
---

# Step 15: Message system

Let's add some narration in our game. In a 3D shooter game, we can imagine the pilot of the ship will discuss with other people: it can be teammates, enemies or even the ship's computer. We will use a message system to display those messages, along with a caption of the speaking character face.

Our message system will be, in a way, similar to the wave system we already have. We will create a xml file containning the messages' content and the time they will be displayed. The lesson will start with that.

## The messages data

### Data in the library project

First, create a `MessageData` class in the library project. It will allow us to create the xml file containing the messages.

```csharp
namespace Tutorial_Data
{
    public class MessageData
    {
        public int id;
        public float time;
        public float duration;
        public string portrait;
        public string message;
    }
}
```

The `id` is the message's id, the `time` is the time when the message will be displayed, the `duration` is the time it will be displayed, the `portrait` is the name of the character face (*none* if nothing to display) to display and the `message` is the text to display.

Let's now create a file `Level0Messages.xml` in our main's project Content folder.  The file will be like this:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<XnaContent>
	<Asset Type="Tutorial_Data.MessageData[]">
		<Item>
			<id>0</id>
			<time>1.0</time>
			<duration>2.0</duration>
			<portrait>pilot</portrait>
			<message>Ok Robot! What do we destroy today?</message>
		</Item>
		<Item>
			<id>0</id>
			<time>3.0</time>
			<duration>3.0</duration>
			<portrait>robot</portrait>
			<message>The motherboard has been overtaken by agressive malignant programs. Unfortunalely, they threaten both the system's data and our corporeal integrity.</message>
		</Item>
		<Item>
			<id>0</id>
			<time>6.0</time>
			<duration>2.5</duration>
			<portrait>robot</portrait>
			<message>I recommand a cautious approach so we can keep the ship's repairs to a minimum.</message>
		</Item>
		<Item>
			<id>0</id>
			<time>8.5</time>
			<duration>2.0</duration>
			<portrait>pilot</portrait>
			<message>What are you useful for if not repairing the ship?</message>
		</Item>
		<Item>
			<id>0</id>
			<time>10.5</time>
			<duration>3.0</duration>
			<portrait>robot</portrait>
			<message>My language subsystems also fit for philosophical dissertation and metaphysical inquiries.</message>
		</Item>
		<Item>
			<id>0</id>
			<time>13.5</time>
			<duration>2.0</duration>
			<portrait>pilot</portrait>
			<message>Not interested. Let's rock!</message>
		</Item>

	</Asset>
</XnaContent>
```

This small dialog introduce the characters and the game setting. Now add the file in the MGCB (with Add / Existing file) and build the content.

### A class representing the messages

In the main project, create a `Message.cs` class. It will be used to represent the messages in the game and help build them and launch their display, in the same fashion as the waves.

```csharp
internal class Message
{
  public int id;
  public float time;
  public float duration;
  public string message;
  public DisplayedCharacter displayedCharacter;

  public Message(int id, float time, float duration, string message, string portrait)
  {
    this.time = time;
    this.id = id;
    this.time = time;
    this.duration = duration;
    this.message = message;
    switch (portrait)
    {
      case "pilot":
        displayedCharacter = DisplayedCharacter.Pilot;
        break;
      case "robot":
        displayedCharacter = DisplayedCharacter.Robot;
        break;
      default:
        displayedCharacter = DisplayedCharacter.None;
        break;
    }
  }

  public void Launch(Game1 game)
  {
      game.DisplayMessage(message, displayedCharacter, duration);
  }
}
```

We will tackle the `DisplayedCharacter` enum and the `Game1.DisplayMessage` function just below.

## A dialog box for our messages

Once we have the message data, we need a place to display it. We will create a `DialogBox.cs` class that will be used to display the messages and the character's face.

Please be sure you have imported in the MBCB all resources needed for the dialog box:
- The dialog box texture (a simple rectangle): `DialogBox.png`
- The character's face textures: `pilot.png` and `robot.png`

You also need to create in the MGCB a new `Arial.spritefont` file, so we can use this font.

### Dialog box content and loading

Our dialog box will be able to:
- Display a written message with a font
- Display a character's face
- Be visible or not
- Stay visible for a certain duration
- Have a background

The lines of text and character's face should be positionned gracefully in the dialog box.

Here is the start of the `DialogBox` class:

```csharp
public enum DisplayedCharacter
{
    None,
    Pilot,
    Robot
}

internal class DialogBox
{
  private string message = "";
  private bool isVisible = false;
  private SpriteFont font;
  private float displayTimer = 0.0f;
  float displayDuration = 3.0f;

  private Texture2D dialogBoxTexture;
  private Texture2D pilotTexture;
  private Texture2D robotTexture;
  private Texture2D portraitTexture;

  const int DIALOG_BOX_X = 20;
  const int DIALOG_BOX_Y = 332;
  const int PORTRAIT_SIZE = 128;
  const int TEXT_START_X = DIALOG_BOX_X + PORTRAIT_SIZE + 50;
  const int MAX_LINE_CHARACTERS = 70;
  const int LINE_SEPARATION = 30;


  public void Load(ContentManager content)
  {
    dialogBoxTexture = content.Load<Texture2D>("DialogBox");
    pilotTexture = content.Load<Texture2D>("PilotDialog");
    robotTexture = content.Load<Texture2D>("RobotDialog");
    font = content.Load<SpriteFont>("Arial");
  }
  ...
```

Most variables are self explanatory. The `portraitTexture` will be the currently displayed character's face. The other texture are just preloaded so we don't have to load them each time we need to display a message.

`MAX_LINE_CHARACTERS` is the maximum number of characters we will display in a line. We will use it to split the message in several lines if needed. `LINE_SEPARATION` is the distance between two lines.

### Displaying and updating the dialog box

The `DisplayMessage` function will used by the `Game1` class to setup the `DialogBox` before displaying it. The `Update` function will be used to manage the `displayTimer` and set the visibility to false when it the display duration is elapsed.

```csharp
  ...
  public void DisplayMessage(string message, DisplayedCharacter character, float duration)
  {
    this.message = message;
    displayDuration = duration;
    isVisible = true;
    switch(character)
    {
      case DisplayedCharacter.Pilot:
        portraitTexture = pilotTexture;
        break;
      case DisplayedCharacter.Robot:
        portraitTexture = robotTexture;
        break;
      default:
        portraitTexture = null;
        break;
    }

  }

  public void Update(double dt)
  {
    if (!isVisible)
    {
      return;
    }
    displayTimer += (float)dt;
    if (displayTimer > displayDuration)
    {
      isVisible = false;
      displayTimer = 0.0f;
    }
  }
...
```

### Drawing the dislog box

Contrary to what we did until now, we will use 2D rendering to draw the dialog box. It will be displayed above our 3D scene - so will be drawn last. With MonoGame, as you already know form the Basic 2D Tutorial, 2D elements are drawn with the `Spritebatch`. That is why the dialog box's `Draw` function and its related functions will use `Spritebatch` as a parameter.

```csharp
  ...
  public void Draw(SpriteBatch spriteBatch)
  {
    if (!isVisible)
    {
      return;
    }
    DrawDialogBox(spriteBatch);
    DrawCharacter(spriteBatch);
    DrawMessage(spriteBatch);
  }

  private void DrawDialogBox(SpriteBatch spriteBatch)
  {
    Color transparentColor = new Color(255, 255, 255, 150);
    spriteBatch.Draw(dialogBoxTexture, new Vector2(DIALOG_BOX_X, DIALOG_BOX_Y), transparentColor);
  }

  private void DrawCharacter(SpriteBatch spriteBatch)
  {
    spriteBatch.Draw(portraitTexture, new Vector2(DIALOG_BOX_X, DIALOG_BOX_Y), Color.White);
  }

  private void DrawMessage(SpriteBatch spriteBatch)
  {
    // Divide the message in lines
    string[] words = message.Split(' ');
    List<string> lines = new List<string>();
    StringBuilder line = new StringBuilder();
    foreach (string word in words)
    {
      if (line.Length + word.Length < MAX_LINE_CHARACTERS)
      {
        line.Append(word + " ");
      }
      else
      {
        lines.Add(line.ToString());
        line.Clear();
        line.Append(word + " ");
      }
    }
    // The last line is added
    lines.Add(line.ToString());
    // Display each line
    for (int i = 0; i < lines.Count; i++)
    {
      spriteBatch.DrawString(font, lines[i], new Vector2(TEXT_START_X, DIALOG_BOX_Y + 20 + i * LINE_SEPARATION), Color.White);
    }
  }
}
```

`DrawDialogBox` draws the dialog box with a transparent color. The `DrawCharacter` function is straightforward, it just draws the character's face.

The `DrawMessage` function is more interesting. It splits the message in words then build a line by adding words in it, keeping the count of the words' characters. When the line is too long, it adds the line to the list of lines and starts a new line. At the end, it must add the line that is currently being built. Finally, it draws each line with a vertical separation.

We can now use our new classes in the `Game1` class.

## Using the message system in the Game1 class

### Loading the message data

We will manage the message data in the `Game1` class the same way we managed the waves. We need some variable and will load the data in the `LoadContent` function. This last function will use a `LoadMessages` function to load the messages from the xml file.

```csharp
public class Game1 : Game
{
  ...
  private DialogBox dialogBox;
  private Tutorial_Data.MessageData[] levelMessagesData;
  private List<Message> messages = new List<Message>();
  private int currentMessage = 0;
  private float messageTimer = 0f;

  ...
  protected override void LoadContent()
  {
    _spriteBatch = new SpriteBatch(GraphicsDevice);

    ...

    dialogBox = new DialogBox();
    dialogBox.Load(Content);
    levelMessagesData = Content.Load<Tutorial_Data.MessageData[]>("Level0Messages");
    messages = LoadMessages(levelMessagesData);
  }

  ...
  private List<Message> LoadMessages(Tutorial_Data.MessageData[] levelMessagesData)
  {
    foreach (Tutorial_Data.MessageData messageData in levelMessagesData)
    {
      messages.Add(new Message(messageData.id, messageData.time, messageData.duration, messageData.message, messageData.portrait));
    }
    return messages;
  }
  ...
}
```

### Updating and drawing the message system

On one hand we will need an `UpdateMessages` function to manage the message schudule. On the other hand, we will update the dialog box it self to display the messages when it is supposed to be visible. We also need a `DisplayMessage` function to forward the message to the dialog box.

```csharp
  ...
  protected override void Update(GameTime gameTime)
  {
    if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
      Exit();

    double dt = gameTime.ElapsedGameTime.TotalSeconds;
    playerAim.Update(dt);
    player.Update(dt);

    UpdateProjectiles(dt);
    UpdateEnemies(dt);
    UpdatePowerUps(dt);
    UpdateWaves(dt);
    UpdateParticleSystems(dt);
    UpdateMessages(dt);

    ground.Update(dt);
    sky.Update(dt);
    dialogBox.Update(dt);

    base.Update(gameTime);
  }

  ...
  private void UpdateMessages(double dt)
  {
    messageTimer += (float)dt;
    if (currentMessage < messages.Count && messageTimer >= messages[currentMessage].time)
    {
      messages[currentMessage].Launch(this);
      currentMessage++;
    }
  }

  public void DisplayMessage(string message, DisplayedCharacter character, float duration)
  {
    dialogBox.DisplayMessage(message, character, duration);
  }
  ...
```

We can now draw the dialog box.

```csharp
  protected override void Draw(GameTime gameTime)
  {
    Color bgColor = new Color(30, 0, 50);
    GraphicsDevice.Clear(bgColor);

    GraphicsDevice.BlendState = BlendState.Opaque;

    ground.Draw(view, projection);
    sky.Draw(view, projection);
    player.Draw(view, projection);

    foreach (Projectile projectile in projectiles)
    {
        projectile.Draw(view, projection);
    }

    foreach (Enemy enemy in enemies)
    {
        enemy.Draw(view, projection);
    }

    foreach (PowerUp powerUp in powerUps)
    {
        powerUp.Draw(view, projection);
    }

    GraphicsDevice.BlendState = BlendState.NonPremultiplied;
    playerAim.Draw(view, projection);
    foreach (ParticleSystem particles in particleSystems)
    {
        particles.Draw(view, projection);
    }

    _spriteBatch.Begin();
    dialogBox.Draw(_spriteBatch);
    _spriteBatch.End();

    base.Draw(gameTime);
  }
```

Test the game now. It works... but it seems the rendering is completely broken. The 3D objects superpose themselves and after a certain time, the ground and sky textures stop repeating. We need to fix that.

### Fixing spritebatch issues

The problem is that we are using the `Spritebatch` to draw the dialog box, it changes some values in the `GraphicsDevice`. More precisely, we are not resetting the `SamplerState` (which manage texture repetition) and `DepthStencilState` (which manage Deapth, so superposition) to their default 3D rendering values. We will fix that by adding two lines of code after the call to the `Spritebatch`.

```csharp
  protected override void Draw(GameTime gameTime)
  {
    ...
    _spriteBatch.Begin();
    dialogBox.Draw(_spriteBatch);
    _spriteBatch.End();

    GraphicsDevice.SamplerStates[0] = SamplerState.LinearWrap;
    GraphicsDevice.DepthStencilState = DepthStencilState.Default;

    base.Draw(gameTime);
  }
```

That's it! Now the dialog box is displayed correctly and the game is back to normal.

## Conclusion

We have now a message system that can be used to display messages in the game. Even if text is not mandatory in games, narration can add a whole layer of involvement to players that are sensible to it. What is super cool is that the timing of the messages is independant from the timing of the waves. It could allow us to have our characters discuss while the player is fighting.

We are nearing the end of this tutorial. In the next steps, we will add a main menu and a game over screen, which will wrap up our entire game.