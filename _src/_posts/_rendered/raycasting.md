---
date: 09/15/2015
title: Unconventional Raycasting
description: Raycasting basics
tags: ['raycasting', 'unity3d']
---
![Alt text](../../../images/raycast-title.png?raw=true "raycast")

# Unconventional Raycasting
If you have not worked on any FPS or any other shooter game until now, it is easy to predict that you would not have needed the raycasting other than conventional way. The commonly used raycasting techniques include the directions up or down, right or left, or forward or backwards.

It can be quite difficult when the angle at which ray to be casted is, say, 45 degrees up, like for flak cannons or SAMs. In this case we can not use simple transform.up with say some +45 or -45 angles. This is possible by specifying the axis. What if the angle we need is something 56 degrees on x-axis and 120 degrees y-axis? And what if you have to design something when there is  change in direction after every frame or a callback implementation. This following tryout demonstrates how easily you can manage to set the angle of the ray where to be casted just by few simple tweaks in your traditional raycasting code.

[The whole project which is in this tutorial can be downloaded from GitHub from here.](https://github.com/talhahasanzia/RaycastTest)

Now first, we need an origin from which ray will be generated, at this moment we are assuming that our origin stays at a fixed position, but we can alter it the way we want. In my case, I have created a scene where a plane is aiming using raycast to determine the aim direction. The origin is the nose tip of the plane from where ray is generated. The plane is in a large box to make the raycasting work. One thing to remember here is that Physics.Raycast returns true only when raycast hits a collider, so we can not do this in open sky. Due to the enclosing box, no matter in which direction the ray is casted, it will be detected. The scene looks like this:

![Alt text](../../../images/ray1.png?raw=true "raycast")


It is not very fancy but it will do the trick. Now we need to have a look at our script. Open file named ‘BasicControl.cs’ in the assets folder of the project and you will see following code:

```
public class BasicControl : MonoBehaviour {
 
public float x = 10, y = 0, z = 0; // angles
 
public GameObject AirCraft;
public GameObject rayCastOrigin;
 
RaycastHit RayHit;
 
void Update ()
 {
 
   if (Physics.Raycast(rayCastOrigin.transform.position, Quaternion.Euler(x, y, z) * transform.forward, out RayHit, Mathf.Infinity))
   {
 
   Debug.DrawLine(rayCastOrigin.transform.position, RayHit.point, Color.red);
 
   }
 
 }
}
```

By setting the access modifier of float x,y,z to public we can change there values from the editor even at runtime. Here x, y and z represents the angles at x , y and z axes respectively. The RaycastHit object tells us the details about the game object on which ray is hit, that is why we have to make sure ray hits something,

```
if (Physics.Raycast(rayCastOrigin.transform.position, Quaternion.Euler(x, y, z) * transform.forward, out RayHit, Mathf.Infinity))
```

Then, if ray collides with an object, RaycastHit collects data of that object which can be used in various ways. Here we are using the position of the object which is being hit by RayHit.point property , it returns x,y,z co ordinates of that object. Rays are not rendered either in scene or game window, so we draw a line from our origin point rayCastOrigin.transform.position to that object.

```
Debug.DrawLine(rayCastOrigin.transform.position, RayHit.point, Color.red);
```

By doing this we will see a line drawn in the path of the ray, which will be visible in scene window. Run the project, click on the Aircraft , you will see the Script component where it shows Basic Control script and its public variables X , Y and Z.

![Alt text](../../../images/ray2.png?raw=true "raycast")



Change the values of these and see how it affects the angle of ray. Now we can adjust the direction of the ray at any angle required. The script can be edited to change the angle at every frame to give it a radar effect.  Or set the ray angle in desired direction and save x, y, z angles for other uses.

![Alt text](../../../images/ray3.png?raw=true "raycast")



[Download project.](https://github.com/talhahasanzia/RaycastTest)


Photo Credit: [Laura Harris (Flickr)](https://www.flickr.com/photos/imajilon/5318340181/)
