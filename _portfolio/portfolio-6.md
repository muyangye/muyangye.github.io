---
title:  "Autonomous Obstacle Avoidance and Line Following Robot"
excerpt: "Programmer<br/><img src='/images/445.jpg'>"
collection: portfolio
---

<p style="text-align: center;"><iframe width="728" height="409.5" src="https://www.youtube.com/embed/I6wIX3dJ1QI" title="Automatic Line Following + Obstacle Avoidance Robot" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></p>

The robot was Pololu 3pi+ 32U4 with Arduino as the microcontroller. I used 2 PID controllers for the task in the video: one that follows the black line using line sensors underneath the robot, one that avoids obstacles (i.e. keeps a certain distance with obstabcles) using sonar attached. The switch of the 2 controllers happen when sonar's staright-facing reading shows too close to something / far enough that shows nothing ahead. Apprarently as shown by the video my kp for line following was too high and I muted the sound because I was talking with my teammate xD.