## Objective
Main objective of this blog post is to give you an idea about how to use Collision Detection without Rigid body in Unity.


## Step 1 : Introduction

Here, just vertical movement is checked but one can check the other directions by modifying the scripts and adding objects to the scene (or changing cube positions).

![](http://www.theappguruz.com/app/uploads/2015/06/collision-detection-without-rigidbody.gif)

Unity is a 3D game engine which comes with built-in physics **PhysX by NVidia**.

Physics simulations are applied to game objects having rigid body attachment.

Mostly it is **used in collision detection**.

Suppose we only want collision detection simulation from physics, then using physics engine only for collision detection may reduce the overall performance. There are two solutions to it:

## Step 2 : Solution

- **Solution - 1**     Make rigid body kinematic and catch trigger events.
- **Solution - 2**     Do not use Rigid body (May be the best solution)!!!!

## Step 3 : Avoid using Rigid Body

**What to use Instead of Rigid body?**

Raycast is a very good option. It is a very versatile facility provided by unity. Moreover it is also cheap. You can cast hundreds of rays per frame without decreasing the performance much. This is a method of casting rays from an origin (provided) towards a direction (provided) and then determining if ray intersects any collider.

We can use this to handle collision detection, by casting rays in both x and y axis to get acknowledgement of the surroundings of the gameobject.

**We will follow the steps given below:**

Get game object direction.
Cast rays.
Determine collision from hit result.

**Simple raycast code looks like this:**

```csharp
if (Physics.Raycast(transform.position, Vector3.forward, 10))
print("There is something in front of the object!");
```

Here, First argument is the origin of ray, second argument is the direction of ray and third argument is the length of ray.

**Unity allows getting the result as shown below:**

```csharp
RaycastHit hitInfo;
if (Physics.Raycast(transform.position, Vector3.down, out hitInfo))
print(“There is something ” + hitInfo.distance +”m away from gameobject”);
```

**E.g.** Let us make a cube to go back when it collides with another cube.

- Place 3 Cubes as below.

![](http://www.theappguruz.com/app/uploads/2015/06/place-cubes.png)

- Add Box Collider to all 3 of them.
- Add CollisionDetector script to the cube which is placed in between the other two cubes (selected in the above image).
- Write the code in CollisionDetector script to make it reflect on collision with the cube above and below.

### 3.1 CollisionDetector.cs

```csharp
using UnityEngine;
using System.Collections;
public class CollisionDetector : MonoBehaviour
{
    public float MovingForce;
    Vector3 StartPoint;
    Vector3 Origin;
    public int NoOfRays = 10;
    int i;
    RaycastHit HitInfo;
    float LengthOfRay, DistanceBetweenRays, DirectionFactor;
    float margin = 0.015f;
    Ray ray;
    void Start ()
    {
        //Length of the Ray is distance from center to edge
        LengthOfRay = collider.bounds.extents.y;
        //Initialize DirectionFactor for upward direction
        DirectionFactor = Mathf.Sign (Vector3.up.y);
    }
    void Update ()
    {
        // First ray origin point for this frame
        StartPoint = new Vector3 (collider.bounds.min.x + margin,transform.position.y,transform.position.z);
        if (!IsCollidingVertically ()) {
            transform.Translate (Vector3.up * MovingForce * Time.deltaTime * DirectionFactor);
    }
}
 
bool IsCollidingVertically ()
{
    Origin = StartPoint;
    DistanceBetweenRays = (collider.bounds.size.x - 2 * margin) / (NOofRays - 1);
    for (i = 0; i<NOofRays; i++) {
        // Ray to be casted.
        ray = new Ray (Origin, Vector3.up * DirectionFactor);
        //Draw ray on screen to see visually. Remember visual length is not actual length.
        Debug.DrawRay (Origin, Vector3.up * DirectionFactor, Color.yellow);
        if (Physics.Raycast (ray, out HitInfo, LengthOfRay)) {
            print ("Collided With " + HitInfo.collider.gameObject.name);
            // Negate the Directionfactor to reverse the moving direction of colliding cube(here cube2)
            DirectionFactor = -DirectionFactor;
            return true;
        }
        Origin += new Vector3 (DistanceBetweenRays, 0, 0);
    }
    return false;
} 
}
```

### 3.2 Explanation

Here **NoOfRays** and **Moving force** are **public variables** so it can be changed at runtime as per the need. Make sure that the moving speed doesn’t exceed the distance between the cube on top and at the bottom.

![](http://www.theappguruz.com/app/uploads/2015/06/collision-detector.png)

**DirectionFactor** is multiplied with movement force and ray direction as it is used to decide the direction. Initially it is set for upward (positive y) direction. As soon as moving cube collides with other cube this direction decider is reversed. Directions can be changed as per the requirement by changing the direction vector. **DirectionFactor** is used only to **reverse the direction**.

