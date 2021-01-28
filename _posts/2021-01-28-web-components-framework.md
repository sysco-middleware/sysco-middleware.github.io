---
layout: post
title: What web components are, and why they have a bright future
categories: front-end
tags: [Web Components, WebRTC, Webpack]
author: JGH153
---

# Intro

This is a blogpost that goes deeper into web components than the youtube video it’s posted alongside. Web components have some issues, and you can see how they can be solved while creating a mini framework here:

<iframe width="560" height="315" src="https://www.youtube.com/embed/AGs7hk0DWP0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Here is a live demo: [web-components-webrtc.web.app](https://web-components-webrtc.web.app/)

And here is the project repo: [github.com/JGH153/web-components-webrtc](https://github.com/JGH153/web-components-webrtc)

# Before Web Components
The web started out with just some simple javascript, but more and more demanding solutions meant we needed an upgrade. The early day javascript did not provide enough features so we started making libraries and frameworks that added the missing features. One of those elements was components. We needed a scalable way to create reusable UI elements and frameworks like Vue, Angular and React came to the rescue with components. This was amazing and really sped up development, but there was a larger problem at hand. You could not mix frameworks. A component made in for example React did not just simply work in another framework. This problem gets even worse as we are getting even more frameworks, and the backlog of older solutions keeps ever growing larger. We need a way to create reusable elements that can be used across all frameworks. We need Web Components.

# The early days
The javascript standard stagnated for a long time before it picked up speed in 2015. It still took some time before web components could be standardized and broadly supported. We even went through an entire version that has now been removed, and it sort of killed a lot of the excitement for web components. The good news is that the new version of web components is here to stay, and has widespread browser support. The lesions learned from the last version are integrated into the new standard, and they are here to stay. 2021 is the year when we can finally use web components in production.

# But what are web components? 
Web components is an umbrella term for several independent web technologies that allow us to create reusable UI elements that can be used anywhere on the web. It is an evolving standard that consists of three core elements: Custom elements, Shadow DOM, and HTML templates. 

Here is a example of a working web component in 25 lines:
[codepen.io/JGH153/pen/abmXLXK](https://codepen.io/JGH153/pen/abmXLXK)

Prefer a video? Here is my beginner friendly introduction to web components:
<iframe width="560" height="315" src="https://www.youtube.com/embed/ae8NF6j_44E" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# HTML templates
HTMl got a new tag: ```<template>```. This is a tag that's not rendered. It only serves as a way to define reusable HTML that can be accessed and used by JavaScript. This is where we write the body/view of our components.

# Shadow DOM
Component encapsulation has long been a key feature provided by modern frameworks. This is where CSS and HTML is separated from the rest of the website. One example is how outside css selectors can affect inside elements and vice versa. It means you don't have to worry about other components styling breaking the one you are making. Modern frameworks just simulate this encapsulation, but Shadow DOM actually fully separates the DOM (and thus styling) from the rest of the application.

# Custom elements
Custom elements are what's holding web components together. It’s a globally available javascript object where to register a web component. You tell it which tag is associated with what javascript class. The javascript class is the logic part of the component, and setup for the component. It gets the content of a HTML template and creates a Shadow DOM with it. The component class is stateful so you can add persistent logic to it.

That's the short version of how web components work. Watch the videos above if you want a deeper look at it. The rest of this blog will dive into challenges, and future possibilities for web components

# Challenges
Web components are great, but they have some significant drawbacks. The biggest is that they are low level. There can be a bit of boilerplate as nothing really comes free with them. A components view will not automatically update when a value changes like in Vue, Angular and React. We have to do that manually. Another challenge is the weak current state of the html templates. We still can’t import other HTML files so we have to add all templates into our index.html file. That's not good. One last problem is that we only can pass numbers and strings to web components. These problems (and more) are currently solvable with webpack (as seen in the video at the top), but the good news is that the standard is evolving to address these issues and more.

# The future of web components
There are a few key features on the horizon for web components that will make it significantly easier to work with. This includes among other elements; HTML Modules, CSS themes and declarative Shadow DOM.

We will gain the ability to **import HTML** files in the future with HTML modules. This will make it significantly easier to use HTML templates as intended without using workarounds. We can then define the template of our components in separate HTML files that are then imported at runtime, like we do with css right now.

Applying some global styles to all web components is almost impossible as the shadow DOM prevents that, as it should. This Makes it hard to globally style applications, but the new **CSS theme** helps to solve this. This pseudoelement could help to allow us target css classes inside closed shadow DOM’s, and thus allow us to add global styles

It is currently only possible to set up a shadow DOM from JavaScript, even if it sometimes makes more sense to set directly on the HTML template tag. This is being addressed with **declarative Shadow DOM** where you could set it directly as:
```<template shadowroot="open">```

Web components will become even better in the near future, but the elements above are not finalized, and might change drastically. Adding something new to Web Components is not like a new feature in React, it is part of the web standard and has to adhere to a higher standard. It has to be able to stand the test of time.

# The framework war
Web components arrived too late. They should have arrived before the three big frameworks gained a foothold. Vue, Angular and React are here to stay and it will be around for decades, even after they have lost all support. This is a problem as what you make on one framework cant be practically used in another. You are locked in. This is really bad when you work at a place with all of them in play and you don’t want to reinvent the wheel each time, in each framework. Web components are truly universal and can be used in every one, but they do not provide as many features as the frameworks. They are more low level.

Angular and Vue have tried to solve this by allowing you to compile to web components, but you will have to drag along the entire framework at the same time. Not a great solution. I think we need more frameworks that use web components at its core, but add the missing features on top. One example of this is Polymer, but it is only a start. 

Another great case for web components over frameworks is material design. This is a design system by Google that started out as a guide, but without an implementation. This was left up to each framework and thus we have Vuetify (Vue), Angular Material and Material-UI (React): The same logic duplicated at least three times. That's insane! Smart people at Google understand this and that is why they are making Material Web Components, re-making all elements one final time as web components:
[github.com/material-components/material-components-web-components](https://github.com/material-components/material-components-web-components)

# Conclusions
Web components are great, but can't survive on its own like they are now. Don’t get me wrong. You can use them today, even make entire applications in them. They work great. However, too many developers are in camp React, Vue and Angular, while web components are left outside in the cold. The frontend world runs on these frameworks, and it will continue like that for a long time. We need a framework built directly on top of web components for it to have a fighting chance, but I believe that would make frontend development significantly better for all of us.

Want to be inspired to create the next web component framework? Check out my video where i make just that and show you how the inner parts of modern frameworks function:
<iframe width="560" height="315" src="https://www.youtube.com/embed/AGs7hk0DWP0" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
