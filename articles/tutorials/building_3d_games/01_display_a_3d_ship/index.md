---
title: "Step 1: Display a 3D ship"
description: Load and display the 3D model of a space ship, using the ContentManager
---

# Introduction

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
    internal class Player
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