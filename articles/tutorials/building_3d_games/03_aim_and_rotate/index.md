---
title: "Step 3: Aim and rotate the player"
description: Use inputs aim toward a displayed target
---

# Step 3: Aim and rotate the player

## Objective

In this chapter, we will create a target that the player will move with the mouse. The player's ship will rotate to aim at the target.

> [!WARNING] Requirements
>
> This chapter is quite mathematical. You need to have understood well [Chapter 1](../01_display_a_3d_ship/index.md) and vector mathematics. You can alse review the explainations about matrices.

## Creating the target's quad

Our target will be a 2D image that will be displayed in the 3D world. This image must have a 3D mesh on which the texture will be applied. We will use a simple quad for this purpose.

A quad is a 2D plane that is composed of two triangles. For optimization purposes, we will have only four vertices for the triangles. Indices will tell which vertices to use to create the two different triangles.

![A quad](images/ch3_quad.png)

> [!IMPORTANT]
>
> The order of the indices determines the orientation of the triangles. If the order is not correct, the triangles will be rendered in the wrong direction, and the texture will not be displayed correctly.

MonoGame allows us to create a mesh by specifying the vertices and the indices that define the triangles. We will create a quad using this in a *Quad.cs* file.

### The Quad data

```csharp
class Quad
{
  private VertexPositionNormalTexture[] vertices;
  private int[] indices;

  private Vector3 origin;
  private Vector3 up;
  private Vector3 normal;
  private Vector3 left;

  public Vector3 upperLeft;
  public Vector3 upperRight;
  public Vector3 lowerLeft;
  public Vector3 lowerRight;

  private BasicEffect effect;
}
```

We use the ``VertexPositionNormalTexture`` class for our vertices. Our quad's vertices will store the position, normal, and texture coordinates. We will also store the indices that define the triangles in our `Quad` class.

[Link to MonoGame VertexPositionNormalTexture documentation](https://docs.monogame.net/api/Microsoft.Xna.Framework.Graphics.VertexPositionNormalTexture.html)

The origin, ``up``, ``normal``, and ``left`` vectors will be used to position the quad in the 3D world. The ``upperLeft``, ``upperRight``, ``lowerLeft``, and ``lowerRight`` vectors will store the position of the quad's corners.

> [!NOTE] About texture coordinates
>
> Texture coordinates, also called UVs, indicate how to map a texture onto a mesh. They are usually in the range [0, 1], where (0, 0) is the upper left corner of the texture and (1, 1) is the lower right corner.
>
> ![Quad's texture coordinates](images/ch3_quad-uv.png)

We will use a `BasicEffect` to render the quad. We will set the effect's texture to the texture we want to apply to the quad.

### Filling the vertices

Let's fill the vertices with the quad's position, normal, and texture coordinates. We will also set the indices for the triangles. This will happen in a `FillVertices` function.

```csharp
private void FillVertices()
{
  Vector2 textureUpperLeft = new Vector2(0.0f, 0.0f);
  Vector2 textureUpperRight = new Vector2(1.0f, 0.0f);
  Vector2 textureLowerLeft = new Vector2(0.0f, 1.0f);
  Vector2 textureLowerRight = new Vector2(1.0f, 1.0f);

  for (int i = 0; i < this.vertices.Length; i++)
  {
      vertices[i].Normal = normal;
  }

  vertices[0].Position = lowerLeft;
  vertices[0].TextureCoordinate = textureLowerLeft;
  vertices[1].Position = upperLeft;
  vertices[1].TextureCoordinate = textureUpperLeft;
  vertices[2].Position = lowerRight;
  vertices[2].TextureCoordinate = textureLowerRight;
  vertices[3].Position = upperRight;
  vertices[3].TextureCoordinate = textureUpperRight;

  indices[0] = 0;
  indices[1] = 1;
  indices[2] = 2;
  indices[3] = 2;
  indices[4] = 1;
  indices[5] = 3;
}
```

### Building the quad

We will create the quad in the constructor of the ``Quad`` class. We will set the ``origin``, ``up``, ``normal``, and ``left`` vectors. We will also set the position of the quad's corners. Finally, we will call the ``FillVertices`` function and store the ``BasicEffect`` we will use to draw.

```csharp
public Quad(Vector3 origin, Vector3 normal, Vector3 up,
    float width, float height, BasicEffect effect)
{
  vertices = new VertexPositionNormalTexture[4];
  indices = new int[6];
  this.origin = origin;
  this.normal = normal;
  this.up = up;

  // Calculate the quad corners
  left = Vector3.Cross(normal, this.up);
  Vector3 uppercenter = (this.up * height / 2) + origin;
  upperLeft = uppercenter + (this.left * width / 2);
  upperRight = uppercenter - (this.left * width / 2);
  lowerLeft = this.upperLeft - (this.up * height);
  lowerRight = this.upperRight - (this.up * height);

  FillVertices();
  this.effect = effect;
}
```

### Drawing the quad

Drawing a mesh set by hand is different from drawing a model. As with the player, we will need a world, view, and projection matrix to draw the quad. The difference is that we  will call the ``GraphicsDevice.DrawUserIndexedPrimitives`` function to draw the quad, and trigger the application of the shader pass. In order to do those last two operations, we will need an access to the ``GraphicsDevice``.

```csharp
public void Draw(GraphicsDevice device, Matrix world, Matrix view, Matrix projection)
{
  effect.World = world;
  effect.View = view;
  effect.Projection = projection;
  foreach (EffectPass pass in effect.CurrentTechnique.Passes)
  {
    pass.Apply();
    device.DrawUserIndexedPrimitives<VertexPositionNormalTexture>(
      PrimitiveType.TriangleList, vertices, 0, 4, indices, 0, 2
    );
  }
}
```

The `GraphicsDevice` contains differents methods related to graphics. Here is a link to the [MonoGame GraphicsDevice documentation](https://docs.monogame.net/api/Microsoft.Xna.Framework.Graphics.GraphicsDevice.html).

## Creating and showing the target

Now the ``Quad`` class is ready, we can create a target that we will be able to move, and that will use the quad to display itself. This target will be contained in a *PlayerAim.cs* file.

In order to draw the player aim's texture, we will need to load it. Add the *Crosshair.png* file into the MGCB and build it. We will then load the texture in the ``Load`` function.

### The PlayerAim class

```csharp
class PlayerAim
{
  private Quad quad;
  private Vector3 position;
  private Quaternion orientation;
  private Matrix world;
  GraphicsDevice device;

  public Vector3 Position
  {
      get { return position; }
  }
}
```

The ``PlayerAim`` class will contain a quad that will be used to display the target. We will also store the position and orientation of the target, and the world matrix and graphics device to draw the quad.

We will also create a property to access the position of the target, so that the player will be able to orientate toward it.

### Setting up the quad and its basic effect

We will use the ``Load`` function to set up our member variables. The position of the target will be set to *(0, 0, -5000)* to place it in front of the camera. We will create a ``BasicEffect`` and set its texture to the *Crosshair* texture. We will then create the quad, orientated towards the player.

```csharp
  public void Load(ContentManager content, GraphicsDevice device)
  {
      position = new Vector3(0, 0, -5000);

      this.device = device;
      BasicEffect effect = new BasicEffect(device);
      effect.VertexColorEnabled = false;
      effect.TextureEnabled = true;
      effect.Texture = content.Load<Texture2D>("Crosshair");
      quad = new Quad(Vector3.Zero, -Vector3.Forward, Vector3.Up, 1000, 1000, effect);
  }
```

### Updating the target

We will update the target in the ``Update`` function. We will use the mouse to move the target. We will then update the world matrix of the quad.

```csharp
  public void Update(double dt)
  {
      MouseState mouse = Mouse.GetState();
      position.X = (mouse.X - device.Viewport.Width / 2) * 10.04f;
      position.Y = (mouse.Y - device.Viewport.Height / 2) * -10.04f;

      world = Matrix.CreateTranslation(position);
  }
```

> [!NOTE]
>
> The player's aim quad won't rotate, so we don't need to set the orientation. The world matrix is simply a translation matrix.

The multiplication by -10.04f is used make the mouse move more dynamic. The multiplication by -10.04f is used to invert the Y axis.

### Drawing the target

We can now call the ``Draw`` function of the quad in the ``Draw`` function of the ``PlayerAim`` class.

```csharp
  public void Draw(Matrix view, Matrix projection)
  {
      quad.Draw(device, world, view, projection);
  }
```

Now we need to create our ``PlayerAim`` object from the ``Game1`` class.

## Managing the PlayerAim

We will now create a ``PlayerAim`` object in the ``Game1`` class. As usual, we will load it in the ``LoadContent`` function, update it in the ``Update`` function, and draw it in the ``Draw`` function.

```csharp
public class Game1 : Game
{
  ...
  private Player player;
  private PlayerAim playerAim;
  ...
  protected override void LoadContent()
  {
    _spriteBatch = new SpriteBatch(GraphicsDevice);

    playerAim = new PlayerAim();
    playerAim.Load(Content, GraphicsDevice);

    player = new Player(playerAim);
    player.Load(Content);
  }

  protected override void Update(GameTime gameTime)
  {
    if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
      Exit();

    double dt = gameTime.ElapsedGameTime.TotalSeconds;
    playerAim.Update(dt);
    player.Update(dt);

    base.Update(gameTime);
  }

  protected override void Draw(GameTime gameTime)
  {
    GraphicsDevice.Clear(Color.CornflowerBlue);

    GraphicsDevice.BlendState = BlendState.Opaque;
    player.Draw(view, projection);

    GraphicsDevice.BlendState = BlendState.NonPremultiplied;
    playerAim.Draw(view, projection);

    base.Draw(gameTime);
  }
}
```

> [!NOTE]
>
> - The player's constructor now takes a PlayerAim object as a parameter.
> - When drawing the PlayerAim, we set the BlendState to NonPremultiplied to ensure transparency.

We will now update the ``Player`` so that it rotates toward its ``PlayerAim``. But before, we need to explain how rotation works in 3D.

## Understanding 3D Rotation

### How to represent rotations in 3D games?

While in 2D games, we can use just one angle to represent rotations, in 3D games, it is not that simple. While 2D objects were just rotated around a single axis, 3D objects can be rotated around three axes: x, y and z. This means that we need to represent rotations in a more complex way than just using angles.

![2D vs 3D rotations](images/ch1_rotations.png)

After some searches, game programmers of old have come to represent 3D rotations all at once with two main mathematical objects: *rotation matrices* and *quaternions*.

*3D matrices* are a table of 4 by 4 numbers that can contain at the same time translation, rotation and scale information. We have already seen them in the first chapter. If we use them for rotations, they have a marvelous feature: they can be multiplied together to apply several rotations at once. This is a very powerful property, but it has a big drawback: they are quite heavy to compute, and they can suffer from *gimbal lock*. Gimbal lock is a problem that occurs when two of the three axes align, causing a loss of one degree of freedom in rotation. This can lead to unexpected behavior in 3D games.

*Quaternions* is a quite abstract mathematical object which is only represented with 4 numbers, and present the same property of being able to be multiplied together (we say *concatenated*) to apply several rotations at once. Additionally, the do not suffer from gimbal lock, and they are more efficient to compute than matrices. Nevertheless, they can just represent rotations, and not translations or scales like matrices do.

The following consensus was finally found. Because they can represent translations, rotations and scales, matrices would be used to contain the final transformation of a 3D object and sent to the GPU to draw objects. Meanwhile, quaternions being more efficient to rotate the 3d objects around, we would use them to execute any rotation during a frame, and then convert them to matrices to apply the final transformation to the 3D object.

We will implement this consensus in our game. We will use a quaternion to represent the orientation of our ship, rotate it with other quaternions, and we will convert it to a matrix to apply the final transformation to the ship's model.

### About Quaternions

#### Quaternions for our game

The orientation variable will hold the rotation of the ship. As stated before, we will use Quaternions to represent rotations.

```csharp
  public void Load(ContentManager content)
  {
    model = content.Load<Model>("Ship");
    position = new Vector3(0, 0.0f, -250.0f);
    orientation = Quaternion.Identity;
    scale = new Vector3(2f, 2f, 2f);
  }
```

The *identity quaternion* is the quaternion that does not rotate the object. It is the equivalent of the zero vector for the `Vector3` class.

#### Quaternions in MonoGame

Usually, in this tutotial's code, in most cases we will create quaternions just before multiplying them to handle rotations.

In this cas we will use the `Quaternion.CreateFromAxisAngle` function. With this function the quaternion will represent a rotation around the axis, by the angle given in radians.

Let's review a case where we create two rotations, one around the x axis and one around the y axis, then multiply (*concatenate*) them together to get a final orientation:

```csharp
var xRotation = Quaternion.CreateFromAxisAngle(Vector3.Right, -MathF.PI / 2);
var yRotation = Quaternion.CreateFromAxisAngle(Vector3.Up, MathF.PI / 4);
orientation = xRotation * yRotation;
// We could have used: Quaternion.Concatenate(xRotation, yRotation);
```

![Rotation concatenation](images/ch1_concatenate-rotations.png)

Sometimes, for a specific reason, we will have a matrix containing the rotation that interests us. In this case, we will be able to create a quaternion from a rotation matrix, using the `Quaternion.CreateFromRotationMatrix` function:

```csharp
return Quaternion.CreateFromRotationMatrix(aim);
```

As stated in this section's introduction, when we have multiplied quaternions together to get a final orientation, we convert this result back to a rotation matrix using the `Matrix.CreateFromQuaternion` function, so we can apply it to our 3D model.

```csharp
var rotationMatrix = Matrix.CreateFromQuaternion(orientation);
```

To learn more about quaternions in MonoGame, here is [MonoGame's Quaternion Documentation](https://docs.monogame.net/api/Microsoft.Xna.Framework.Quaternion.html).

#### Why Not Just Use Angles?

Before discussing quaternions, it's important to understand why we don't just use Euler angles (pitch, yaw, roll) with rotation matrices. While angles are intuitive, they have significant limitations in 3D graphics:

- Gimbal Lock: When rotations on one axis cause another axis to align, losing a degree of freedom, rotation becomes unpredictable.
- Interpolation Problems: Smoothly transitioning between rotations is difficult with angles
- Numerical Stability: Accumulated errors can cause issues over time

#### What is a Quaternion?

Now that we know how to use them, let's understand further what a quaternion is.

Mathematically, a quaternion is a four-dimensional number represented as:

$$
q = w + xi + yj + zk
$$

Where:

- *w* is the real component
- *x*, *y*, *z* are imaginary components
- *i*, *j*, *k* are special operators with properties like $i² = j² = k² = ijk = -1$

Yes, this last property feels a bit weird. You may have learned that real numbers (numbers from $\mathbb{R}$), when squared, cannot be negative - so their squared values cannot be equal to -1. Actually, there are other sets of numbers than real numbers. You might have heard about complex numbers (in $\mathbb{C}$), which are numbers that can be represented as a + bi, where a and b are real numbers and i is the imaginary unit, with the property that i² = -1. Quaternions are an extension of complex numbers, living in a 4 dimensions space called the Hamiltonian space ($\mathbb{H}$).

In code, a quaternion is stored as a vector with 4 dimensions (x, y, z, w).

> [!NOTE]
>
> You cannot represent yourself a number in 4 dimensions? That is normal: it is a quite abstract mathematical object. Actually, you do not need to understand the underlying mathematics to use quaternions. You can consider it as a tool to ease rotation computations.

#### Mathematical properties

**1. Quaternion.Identity:** Represents "no rotation" (like in your code)

```csharp
Quaternion.Identity = Quaternion(0, 0, 0, 1)
// Here, w = 1, x = 0, y = 0, z = 0. 
// w is at the end of the constructor.
```

**2. Normalization:** Like vectors, quaternions must be normalized for rotation. Using a non-normalized quaternion can cause unexpected rotations.

$$
|q| = \sqrt{w² + x² + y² + z²}
$$

$$
\hat{q} = \frac{q}{|q|}
$$

```csharp
orientation.Normalize();
```

**3. Quaternion Multiplication:** Combines rotations.

This is the key to quaternions' efficiency, because it is way quicker to multiply quaternions than to multiply rotation matrices.

$$
q_1 * q_2 = (w_1w_2 - x_1x_2 - y_1y_2 - z_1z_2) + (w_1x_2 + x_1w_2 + y_1z_2 - z_1y_2)i + (w_1y_2 - x_1z_2 + y_1w_2 + z_1x_2)j + (w_1z_2 + x_1y_2 - y_1x_2 + z_1w_2)k
$$

MonoGame allows us to multiply quaternions:

```csharp
orientation = orientation * Quaternion.CreateFromAxisAngle(Vector3.Up, MathHelper.Pi);
```

**4. Converting to a Rotation Matrix:** When we have computed the final orientation of our object with quaternions, we can convert it to a rotation matrix to apply it to our object.

We will discuss matrices just below.

```text
Matrix rotationMatrix = Matrix.CreateFromQuaternion(orientation);
```

**Application of Quaternions in Game Development:** Quaternions are used for:

- Representing object orientation (like your ship)
- Smooth rotation interpolation (SLERP - Spherical Linear Interpolation)
- Camera control
- Character animation

Again, this list is far from exhaustive! But for now, that's ok. We can come back to our game and use Quaternions to rotate our player toward the aim target.

## Orienting the Player toward the PlayerAim

First, we need to add a ``PlayerAim`` member variable to the ``Player`` class. It will be passed to they player in a new constructor.

```csharp
  private PlayerAim playerAim;

  public Player(PlayerAim playerAim)
  {
      this.playerAim = playerAim;
  }
```

### The HandleAiming function

In the Player class, we will create an ``HandleAiming`` function that will rotate the player toward the target. We will then call this function in the ``Update`` function.

```csharp
  private void HandleAiming()
  {

  }

  public void Update(double dt)
  {
    HandlingInput(dt);
    HandleAiming();

    world = Matrix.CreateScale(scale) * Matrix.CreateFromQuaternion(orientation) * Matrix.CreateTranslation(position);
  }
```

### Calculating the rotation to follow the aim

The general idea here will be to compute the direction between the player and the aim, then create a rotation matrix from this direction, then convert this matrix into the quaternion.

We can get the direction by subtracking the aim's position from the player's position. We will then normalize this direction vector to get a unit vector.

![Player aiming at the target](images/ch3_aiming.png)

To orientate the player, we will build with our little hands an orientation matrix. It is not the simplest solution here but it will be useful for you to help understand that a matrix represent a coordinate system relatively to an object. In the end, we will have a matrix that represents the coordinate system of the player, oriented toward the target.

Now we have a normalized vector toward the target, we build a perpendicular vector to this vector, by executing a cross product between our normalized direction vector and the world's up vector. We normalize the result. Finally, we create a last normalized perpendicular vector - this time perpendicalar to both the direction and the second vector. Those three normalized vector create a coordinate system specific to the player's orientation. The following diagram reprensents the coordinate system we just created:

![Orientation matrix](images/ch3_orientation-matrix.png)

We can use the coordinates of those three vectors to create a matrix that represent the transformation leading to this coordinate system. We then create a quaternion from this matrix.

Then, in order to create this rotation matrix, and if we consider the direction vector as a forward vector, we need to create two other vectors: the right vector (xAxis) and the up vector (yAxis), relative to our forward vector. We will then create a matrix from these three vectors, by setting the matrix's value by hand.

When we have the orientation matrix, we create a quaternion from it and set it as the player's orientation.

```csharp
  private void HandleAiming()
  {
    Vector3 direction = playerAim.Position - position;
    direction.Normalize();

    Vector3 xAxis = Vector3.Cross(Vector3.Up, direction);
    xAxis.Normalize();

    Vector3 yAxis = Vector3.Cross(direction, xAxis);
    yAxis.Normalize();

    Matrix aim = Matrix.Identity;
    aim.M11 = xAxis.X;
    aim.M21 = yAxis.X;
    aim.M31 = direction.X;

    aim.M12 = xAxis.Y;
    aim.M22 = yAxis.Y;
    aim.M32 = direction.Y;

    aim.M13 = xAxis.Z;
    aim.M23 = yAxis.Z;
    aim.M33 = direction.Z;

    orientation = Quaternion.CreateFromRotationMatrix(aim);
  }
```

Everything is ready! Run the game and rotate the player toward the target while moving the ship with the keys and the mouse.

![Chapter 3 result](images/ch3_final-screen.png)

## Conclusion

In this chapter, we created a target that the player can aim at using the mouse. The player rotates to face the target. We also learned how to create a quad mesh and apply a texture to it.

In the next chapter, we will add a shooting mechanic to the player, allowing it to shoot in the direction of the target. We will also create a projectile class to handle the bullets' behavior and rendering.
