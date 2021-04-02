---
title:  "Computational Fluid Dynamics: There and Back Again"
layout: single
author_profile: true 
excerpt: "A brief computational fluid dynamics validation of a gas-liquid heterogenous reactor, adapted from my BEng Design Project thesis."
header:
  overlay_image: /assets/images/CFDPATH.PNG
  overlay_filter: rgba(140,140,140, 0.5)
  caption: "Particle tracking within reactor"
classes: wide
---

## Case Study
For my BEng design project I was tasked with the design of a gas liquid reactor. Specifically pertaining to the reaction of carbon monoxide, methanol, and ethylene to produce methyl propionate. Two gases and a liquid react to form a liquid. 

Having considered the mass and energy balance, the recycle structure, catalyst choice and amount, one thing remained: _validating my assumptions_. Namely, **is my reactor _actually_ well mixed?**.

And so began my 'brief' foray into computational fluid dynamics.
   
## Modelling the reactor

Using an almost extraordinary number of correlations from the 70s as well as some extreme patent detective-work I settled on some reactor and agitator dimensions. Now all I had to do was just plug them into ANSYS Fluent and job done. Oh... it's [$5,000](https://catalog.ansys.com/product/5b3bc6857a2f9a5c90d32e78/mixing-guided-proc). 

So I began the long journey of modelling my reactor geometry in ANSYS Design Modeller. Whilst there was a decent learning curve for CAD software, I gained a real appreciation for the parametric modelling philosophy. With a bit of effort I was able to very easily able change my reactor dimensions, number of baffles, and mixing arangements.  

<p align="center">
<img src="/assets/images/impeller.PNG" width="45%"> <img src="/assets/images/imp side.PNG" width="45%">
</p>

I worked from some technical drawings to model my offset A315 impellers (chosen to maximise gas contact), this was probably the trickiest bit but most satisfying. The idea of a 3D CAD model really being a set of specific parametric instructions was growing on me.

<p align="center">
<img src="/assets/images/reactor_model.PNG" width="50%">
</p>

## Meshing

When separating the continuous space within my reactor to finite elements there are a number of techniques available. I could take advantage of the axial symmetry, refine my mesh, optimise the number tetrahedrons, identify low fidelity regions. I'm sure there are entire theses on the subject. But to be honest... I just let ANSYS mesh my internals automatically. Am I proud, no, but I'm going to place my faith in the ANSYS corporation and hope that the person coding the automatic meshing implementation was having a good day. In hindsight it wasn't the absolute best but it certainly did the job. File under _future work_. 

<p align="center">
<img src="/assets/images/mesh.PNG" width="50%">
</p>


## Fluid &rarr; Fluids &rarr; Fluid

Here I encounter my first real issue. My reactor is roughly 5m wide, and I want to see if gas bubbles are well mixed, except my bubbles of gas are on the order of _millimetres_. 

Without entering the field of multiscale modelling, which I wasn't willing to do because I still had to write a lengthy and thoroughly interesting section on the heirarchy of process alarms around my reactor, there was little I could do here.

I conceded to making an approximation in the validation of my original approximation, by just simulating the liquid within my reactor, and introducing some Lagrangian fluid particles entering from the gas sparger. 
I did however get the liquid properties directly from my Aspen HYSYS flowsheet (using an accurate and validated UNIQUAC thermodynamic model), the only redeeming aspect of this section. 

## Results 

Initially, when using two impellers the same size I was gaining decent overall fluid velocity, however there was a large stagnant region where the fluid from the upper and lower sections of the reactor meet.

<p align="center">
<img src="/assets/images/2SAME2.PNG" width="50%">
</p>

This motivated me to reduce the size of the upper impeller slightly in a bid to reduce the area of this stagnant zone, and I believe it worked. 
I was actually very pleased with this observation because it's exactly the same modification that Lucite detail in their patent of the reactor. 

You can see above the results of my sloppy meshing procedure, the dark blue areas of no fluid flow around the sharp corner of the rotating impeller fluid region. 

I changed my impeller dimensions and my reactor was showing signs of becoming well mixed. 
After a brief mechanical design validation in ANSYS Static, I was able to produce a reactor schematic. 

<p align="center">
<img src="/assets/images/drawing.png" width="50%">
</p>


## Well Mixed?

<p align="center">
<img src="/assets/images/CFDPATH.PNG" width="50%">
</p>

Going back to the very first purpose of this, was my reactor really well mixed? 
Yes and no. I did find that Lagrangian particles were able to become well dispersed within the reactor, however (as expected) there were very much two regions within the reactor. One controlled by the upper impeller and one by the lower. 
Using this information I could have re-iterated my design process and remodelled my reactor as two CSTRs in series/parallel, however unfortunately all good things must come to an end. 

Learning CFD in this context was quite enjoyable, it provided me with a viewpoint of parametric CAD modelling, rotating fluid meshes, fluid-solid interactions, plotting results, turbulence models. Overall I'm very pleased with the results!

## Authors

* **Tom Savage** - *Initial work* - [TomRSavage](https://github.com/TomRSavage)
