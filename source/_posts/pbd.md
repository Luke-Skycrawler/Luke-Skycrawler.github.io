---
title: Position-Based Dynamics
date: 2021-11-17 17:58:13
tags: 
- glut
thumbnail: 
- /img/pbd/pbd.jpg
---

A simple Position-based Dynamics implementation based on glut. <!-- more --> My starter project in physical simulations, which is assigned to me as an interview task. No clamping or self-collision is added, so the cloth folds eccentrically and never stops shaking. Other problems include the sphere always dropping to the left, probably because the Gaussian-Sedel iteration always starts from the right side of the cloth mesh, which introduces an unbalanced impact. Only the vertices are tested in the collision detection, and a little lift is added to the sphere to avoid visual artifacts. Visit [github](https://github.com/Luke-Skycrawler/pbd) for full code.

{% raw %}
<video src="/videos/demo.mp4" type='video/mp4' controls='controls' width='100%' height='100%'>
</video>
{% endraw%} 