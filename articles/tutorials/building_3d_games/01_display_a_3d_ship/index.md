---
title: "Step 1: Display a 3D ship"
description: Load and display the 3D model of a space ship, using the ContentManager
---

# Step 1: Display a 3D ship - Introduction

## Objective

In this first tutorial, we will see how to load and display the space ship that will serve to represent the player in our game. We will position it in the game, setup a camera to watch it and integrate that in the Game1 class. This will allow us to review 3D mathematics concept and their API in Monogame.

## Requierements

You are supposed to have read lessons 1, 2, 3 and 6 of the Monogame's 2D basic tutorial. You should know about the Game1 class, its LoadContent, Update and Draw functions and the use of the GameTime parameter in Update and Draw.

# The Player class

Create a Player class. I'll walk you through the 3D game programming concepts in this Player.cs file, focusing on the mathematical aspects while keeping things approachable for beginners. We will for now create the different member variables of this class, and iits Load and Update function.

Create the class squeletton in such a way:

```csharp
class Player
{
    private Model model;
    private Vector3 position;
    private Quaternion orientation;
    private Matrix world;

    public void Load(ContentManager content)
    {

    }

    public void Update(double dt)
    {

    }
}
```

## Loading the Model

### Add the model in MGCB

Double click on the Content/Content.mgcb file. If monogame is correctly installed, the MGCB window should open. Right click on the Content icon, Add, Existing Item, and select the Ship.fbx file and its ShipTexture.png files from the resource folder you have unzipped. Accept to copy those files in the Content folder.

The MGCB should have automatically selected the right Importer and Processor for the files you have chosen. Click on the Build icon, shut down MGCB.

### Load the model in Player class

The model will be stored in the model variable.

```csharp
    private Model model;
```

We will fill this variable by loading the file fron the content manager:

```csharp
    public void Load(ContentManager content)
    {
        model = content.Load<Model>("Ship");
    }
```

## Positionning the player's ship

We will position the player ship thanks to a Vector3, to manage its position and a Quaternion, to manage rotations. Those two variables will allow us to compute a world transform Matrix, which will hold the final space coordinates of our ship.

```csharp
    class Player
    {
        private Model model;
        private Vector3 position;
        private Quaternion orientation;
        private Matrix world;
        ...
```

### The position vector

The Vector3 position contains the 3D world position of out ship. Before diving deeper, let's understand how 3D space works in games:

Vector3 contain 3 coordinates, x, y, z, along 3 axis:
- X-axis: Runs horizontally (left to right)
- Y-axis: Runs vertically (up and down)
- Z-axis: Runs depth-wise (forward and backward)

We decide a special point is the origin: the point where x, y and z coordinates are zero.

In our game, the initial of our ship will be Vector3(0, 0.0f, -250.0f).

```csharp
    public void Load(ContentManager content)
    {
        model = content.Load<Model>("Ship");
        position = new Vector3(0, 0.0f, -250.0f);
    }
```

This places the ship:

At the center horizontally (x = 0)
At the bottom/middle vertically (y = 0)
250 units into the screen/away from the camera (z = -250)

### About Vector3 in Monogame

#### Basic Definition
A Vector3 represents either:
- A point in 3D space (x, y, z coordinates)
- A direction with magnitude (like velocity or force)

A point in 3D space (x, y, z coordinates)
A direction with magnitude (like velocity or force)

#### Mathematical Properties

**1. Addition/Subtraction:** When you add two vectors, you add their components.
```
(x1, y1, z1) + (x2, y2, z2) = (x1+x2, y1+y2, z1+z2)
```

In Monogame, Vector3 can be added:
```csharp
var a = new Vector3(0, 1, 2);
var b = new Vector3(3, 4, 5);
Vector3 result = a + b;
```

If you can add them, you can substract them. But there is no equivalent for the usual multiplication between two Vector3.

**2. Magnitude (Length):** The distance from origin to the point.
```
|a| = √(x² + y² + z²)
```

Monogame provides you with a way to directly magnitude from a vector. You can also get the squared magnitude (magnitude multiplied by itself), that is useful for more optimal computing.

```csharp
Vector3 distanceFromOrigin = position.Length();
Vector3 distanceFromOriginSquared = position.LengthSquared();
```

**3. Normalization:** Creating a unit vector (length of 1) in the same direction.
```
v̂ = v/|v| = (x/|v|, y/|v|, z/|v|)
```

Usually, we want a normalized vector when we want a direction with no length information. Normalizing will set the length of the vector to 1. Monogame provides us with a way to easily do this operation:

```csharp
Vector3 directionFromOrigin = position - Vector3.Zero;
directionFromOrigin.Normalize();
```

In this specific case, we do not need to substract the zero vector to our position vector, but it allowed us to introduce the fact that a direction between two points in the normalized difference between end point's and starting point's positions ;)

**4. Dot Product:** Measures how parallel two normalized vectors are.
```
a·b = ax*bx + ay*by + az*bz
```
- If a·b = 0, vectors are perpendicular
- If a·b > 0, angles between vectors is less than 90°
- If a·b < 0, angle is greater than 90°

This operation is super useful in videogames. The Monogame's Vector3 class gives us a way to easily compute dot products:

```csharp
var a = new Vector3(0, 1, 2);
var b = new Vector3(3, 4, 5);
a.Normalize();
b.Normalize();
float dotProduct = Vector3.Dot(a, b);
```

Do not forget to normalize a and b!

**5. Cross Product:** Creates a new vector perpendicular to both inputs.
```
a×b = (ay*bz - az*by, az*bx - ax*bz, ax*by - ay*bx)
```

The cross product between two vectors give a vector that is perpendicular to the plane defined by the two input vectors. This is super useful to compute normal vectors, which are used in lighting computations. Monogame provides us with a way to easily compute cross products:

```csharp
var a = new Vector3(0, 1, 0);   // Up
var b = new Vector3(1, 0, 0);   // Right
Vector3 crossProduct = Vector3.Cross(a, b);
```

In this case, the cross product will give us the forward vector, which is perpendicular to the up and right vectors. The two vectors we have chosen do not have to be normalized, except if you want the cross product result to be normalized.

** Application of Vector3 in Game Development:** Vectors are used for a lot of 3D calculations in games:
- Object positions (like the ship's position in the code)
- Movement directions and velocity
- Forces (gravity, thrust)
- Surface normals (for lighting calculations)

### Orientation with Quaternions

The orientation variable will hold the rotation of the ship. We will use Quaternions to represent rotations.

```csharp
    public void Load(ContentManager content)
    {
        model = content.Load<Model>("Ship");
        position = new Vector3(0, 0.0f, -250.0f);
        orientation = Quaternion.Identity;
    }
```

The identity quaternion is the quaternion that does not rotate the object. It is the equivalent of the zero vector for the Vector3 class.

Quaternions are a way to represent rotations in 3D space. They are more efficient than matrices for rotations, and they do not suffer from gimbal lock. We will see in further lessons how to use them to rotate our ship.

### Quaternions in Monogame

#### Why Not Just Use Angles?
Before discussing quaternions, it's important to understand why we don't just use Euler angles (pitch, yaw, roll):

- Gimbal Lock: When rotations on one axis cause another axis to lose a degree of freedom
- Interpolation Problems: Smoothly transitioning between rotations is difficult with angles
- Numerical Stability: Accumulated errors can cause issues over time

#### What is a Quaternion?

A quaternion is a four-dimensional number system represented as:
```
q = w + xi + yj + zk
```

Where:
- w is the real component
- x, y, z are imaginary components
- i, j, k are special operators with properties like i² = j² = k² = ijk = -1

In code, a quaternion is often represented as a vector with 4 dimensions (x, y, z, w).

#### Mathematical properties

**1. Quaternion.Identity:** Represents "no rotation" (like in your code)
```
Identity = (0, 0, 0, 1)
```

**2. Normalization:** Like vectors, quaternions must be normalized for rotation. Using a non-normalized quaternion can cause unexpected rotations.
```
|q| = √(w² + x² + y² + z²)
q̂ = q/|q|
```

```csharp
orientation.Normalize();
```

**3. Quaternion Multiplication:** Combines rotations. This is the key to quaternions' efficiency, because it .
```
q1 * q2 = (w1w2 - x1x2 - y1y2 - z1z2) + (w1x2 + x1w2 + y1z2 - z1y2)i + (w1y2 - x1z2 + y1w2 + z1x2)j + (w1z2 + x1y2 - y1x2 + z1w2)k
```

Monogame allows us to multiply quaternions:

```csharp
orientation = orientation * Quaternion.CreateFromAxisAngle(Vector3.Up, MathHelper.Pi);
```

**4. Converting to a Rotation Matrix:** When we have computed the final orientation of our object with quaternions, we can convert it to a rotation matrix to apply it to our object.

We will discuss matrices just below.
```
Matrix rotationMatrix = Matrix.CreateFromQuaternion(orientation);
```

**Application of Quaternions in Game Development:** Quaternions are used for:
- Representing object orientation (like your ship)
- Smooth rotation interpolation (SLERP - Spherical Linear Interpolation)
- Camera control
- Character animation

### Inserting the player in the world with the world matrix

#### What is the World Matrix?

A Matrix in an algebraic structure (a mathematical tool) that is used to represent linear applications, that is to say, the transformation of a vector in an other vector. In 3D graphics, matrices are used to represent specific applications, named transformations, like rotation, scaling, and translation.

Each vertex of a 3D object is represented by a Vector3, with values set from its origin (the point (0, 0, 0) in blender for instance). To insert this 3d object in the game world, where the object is probably set at a specific position, rotation and scale (size change), we need to convert each vertex coordinate from the object space the world space. To achieve that, we multiply the vertex by a matrix, called the world matrix, that combine translation, rotation and scale operations. The result is a new Vector3, which is the transformed vertex.

#### Computing the World Matrix

In our code, our player stores a World matrix:
```csharp
    class Player
    {
        private Model model;
        private Vector3 position;
        private Quaternion orientation;
        private Matrix world;
        ...
```

Because position and rotation will constantly change, we need to update the world matrix every frame. We will compute this matrix in the Update function:

```csharp
    public void Update(double dt)
    {
        world = Matrix.CreateFromQuaternion(orientation) * Matrix.CreateTranslation(position);
    }
```

As we have seen, we can create a rotation matrix from the orientation quaternion, and a translation matrix from a position vector. We can multiply those two matrices to get the final world matrix. In this case we do not use a scale matrix, but we will later.

### Matrices in monogame

A matrix in 3D graphics is typically a 4×4 grid of numbers that can represent a transformation. In the MonoGame framework, Matrix is this structure.

#### Translation, rotation, and scale matrices

**1. Translation Matrix:** Moves objects in 3D space

```
[1 0 0 X]
[0 1 0 Y]
[0 0 1 Z]
[0 0 0 1]
```
Where X, Y, Z are the distances to move along each axis.

MonoGame provides a way to create a translation matrix:

```csharp
Matrix translationMatrix = Matrix.CreateTranslation(10, 20, 30);
```

**2. Rotation Matrix:** Rotates objects. The matrix varies by axis.
Rotation around X-axis:
```
[1  0       0       0]
[0  cosθ    -sinθ   0]
[0  sinθ    cosθ    0]
[0  0       0       1]
```
Where θ is the angle of rotation.

Rotation around Y-axis:
```
[cosθ   0   sinθ    0]
[0      1   0       0]
[-sinθ  0   cosθ    0]
[0      0   0       1]
```

Rotation around Z-axis:
```
[cosθ   -sinθ   0   0]
[sinθ   cosθ    0   0]
[0      0       1   0]
[0      0       0   1]
```

MonoGame provides a way to create a rotation matrix:

```csharp
Matrix rotationMatrix = Matrix.CreateFromAxisAngle(Vector3.Up, MathHelper.Pi);
```

Rotation matrices can be combined to create a rotation matrix that rotates around multiple axes. Nevertheless, quaternions are more efficient for this kind of operation and avoid the gimbal lock problem. As stated above, it is better to use quaternions to create complex rotations, then go back to matrices when needed.

**3. Scale Matrix:** Changes the size of objects

```
[X 0 0 0]
[0 Y 0 0]
[0 0 Z 0]
[0 0 0 1]
```
Where X, Y, Z are the scaling factors.

MonoGame provides a way to create a scale matrix:

```csharp
Matrix scaleMatrix = Matrix.CreateScale(2, 3, 4);
```

#### Matrix multiplication and the World matrix

Multiplying two matrices is a way to combine their transformations. There is a mathematical rules that are important to know: order is important!

```
A * B != B * A
```

The standard order is: Scale → Rotate → Translate. Not respecting this order can lead to unexpected results. For instance, if you translate before rotating, the object will rotate around the world's origin, not around its center.

In the case of the world matrix, we combine the translation, rotation, and scale matrices to transform a vertex from object space to world space.

The world matrix is the matrix that combines translation, rotation and scale matrices to transform a vertex from the object space to the world space. It is the matrix that is used to insert the object in the game world.

In our code, and because we are not scaling the we will create it from the position and orientation of the player:

```csharp
    public void Update(double dt)
    {
        world = Matrix.CreateFromQuaternion(orientation) * Matrix.CreateTranslation(position);
    }
```

That's it for the Player class. The only missing part is drawing, but we will get back to it later. We will now integrate our player in the Game1 class.

# Using the Player in the Game1 class

We will now integrate the player in the Game1 class. We will load the player in the LoadContent function, update it in the Update function. The draw part will have a chapter on its own.

## Load the player

First we need to add a member variable to the Game1 class to store the player:

```csharp
    class Game1 : Game
    {
        private GraphicsDeviceManager _graphics;
        private SpriteBatch _spriteBatch;
        private Player player;
        ...
```

Now we can create the player in the LoadContent function:

```csharp
    protected override void LoadContent()
    {
        _spriteBatch = new SpriteBatch(GraphicsDevice);

        player = new Player();
        player.Load(Content);
    }
```

Note that we pass the ContentManager (the Content variable) to the Load function of the player. This is because the player needs to load the model from the content manager. The Content variable comes from the Game class, from which Game1 inherit, and is automatically initialized by the Game class.

## Update the player

We will now update the player in the Update function. As you may have remarked in the Player class, the Update function takes a double parameter, dt, which is the time elapsed since the last frame. We need to create this parameter in the Update function of the Game1 class, and pass it to the Update function of the player.

```csharp
    protected override void Update(GameTime gameTime)
    {
        if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
            Exit();

        double dt = gameTime.ElapsedGameTime.TotalSeconds;
        player.Update(dt);

        base.Update(gameTime);
    }
```

Ok, we have a player in our game, that is updated every frame. We will now see how to draw it.

# Basic 3D drawing with Monogame

## The view and projection matrices

We have seen that the world matrix is used to transform a vertex from the object space to the world space. But we cannot stop here. First, we need to see the world from the point of view of a camera. Then we need to project the 3D world "filmed" by this camera to your 2D screen. To achieve that, we need two other matrices: the view matrix and the projection matrix.

The view matrix is used to transform a vertex from the world space to the camera space. Monogames provides us with a way to create a view matrix from a position, a target and an up vector. The position is the position of the camera, the target is the point the camera is looking at, and the up vector is the direction that is considered as up.

```csharp
private Matrix view = Matrix.CreateLookAt(new Vector3(0, 0, 100), new Vector3(0, 0, 0), Vector3.UnitY);
```

The projection matrix is used to transform a vertex from the camera space to the screen space. There are usually two ways to create a projection matrix: create a pespective projection matrix or an orthographic projection matrix. The perspective projection matrix is used to create a perspective effect, where objects that are far away are smaller than objects that are close. The orthographic projection matrix is used to create an isometric effect, where objects that are far away are the same size as objects that are close.

In our case, we will use a perspective projection matrix. Monogame provides us with a way to create a perspective projection matrix from a field of view, an aspect ratio, a near plane and a far plane. The field of view is the angle of the camera's field of view, the aspect ratio is the ratio of the screen's width to the screen's height, the near plane is the distance from the camera to the near clipping plane, and the far plane is the distance from the camera to the far clipping plane.

The near and far clipping planes are used to clip objects that are too close or too far from the camera. Objects that are too close or too far are not drawn. This is useful to improve performance, because objects that are not drawn are not processed by the GPU. The shape of the clipping volume is known as a frustum, which is a pyramid with the top cut off.

Monogame provides us with a way to create a perspective projection matrix (and also an orthographic projection matrix, but we won't use it in this tutorial):

```csharp
private Matrix projection = Matrix.CreatePerspectiveFieldOfView(MathHelper.ToRadians(45), 800f / 480f, 1f, 10000f);
```

We update the Game1 class to store those two matrices:

```csharp
class Game1 : Game
{
    private GraphicsDeviceManager _graphics;
    private SpriteBatch _spriteBatch;

    private Player player;
    private Matrix view = Matrix.CreateLookAt(new Vector3(0, 0, 100), new Vector3(0, 0, 0), Vector3.UnitY);
    private Matrix projection = Matrix.CreatePerspectiveFieldOfView(MathHelper.ToRadians(45), 800f / 480f, 1f, 10000f);
    ...
```

## Drawing the player with the world, view, projection matrices, and the BasicEffect

We will now draw the player by defining its Draw function. We need to use the world, view, and projection matrices to transform the player's model from the object space to the screen space.

The combination of the world, view and projection matrices is called the world-view-projection matrix, or model-view-projection matrix (MVP matrix). In 3D engines and AOIS, this matricx is computed in a program that run on the GPU and is called a shader. More specifically, it is needed to have at least a vertex shader to compute this matrix, and a fragment shader to compute the color of the pixel on the screen once we know the coordinates of each vertex on the screen.

In Monogame, the BasicEffect class can play the role of both the vertex and fragment shader. It is a class that is used to draw 3D models with a basic effect, like a single color or a texture. We will use the BasicEffect class to draw the player's model.

Here is the code for the Player's class Draw function:

```csharp
    public void Draw(Matrix view, Matrix projection)
    {
        foreach (ModelMesh mesh in model.Meshes)
        {
            foreach (BasicEffect effect in mesh.Effects)
            {
                effect.World = world;
                effect.View = view;
                effect.Projection = projection;
            }

            mesh.Draw();
        }
    }
```

As you can see, we loop through the model's meshes, and for each mesh, we loop through the basic effects. We set the world, view, and projection matrices of the effect to the world, view, and projection matrices of the player. We then draw the mesh.

We will make a further use of the BasicEffect later in the tutorial. For now, let's integrate the player's Draw function in the Game1 class.


## The GraphicsDevice

In the Game1 class, you may have remarked the GraphicsDeviceManager member variable. The GraphicsDeviceManager is used to create the GraphicsDevice, which is used to draw the game. The GraphicsDevice is created when the Game class is initialized, and is accessible through the GraphicsDevice property of the Game class.

We create the GraphicsDeviceManager in the Game1 constructor:

```csharp
    public Game1()
    {
        _graphics = new GraphicsDeviceManager(this);
        Content.RootDirectory = "Content";
        IsMouseVisible = true;
    }
```

You have noticed the GraphicsDevice is used to initialize the spriteBatch in the LoadContent function:

```csharp
    protected override void LoadContent()
    {
        _spriteBatch = new SpriteBatch(GraphicsDevice);

        player = new Player();
        player.Load(Content);
    }
```

We won't use the spriteBatch to draw until the UI part of the tutorial, but it worth mentioning.

## Drawing the player and using the GraphicsDevice

Now we have everything ready, we will now integrate the player's Draw function in the Game1 class. We will also use the GraphicsDevice to clear the screen and draw the player.

```csharp
    protected override void Draw(GameTime gameTime)
    {
        GraphicsDevice.Clear(Color.CornflowerBlue);

        player.Draw(view, projection);

        base.Draw(gameTime);
    }
```

If you launch the game, you should see the player's model displayed on the screen. The player's model should be at the center of the screen, and should be facing the camera. We will orientate it to the far plane of the camera and make it move in the next lesson!