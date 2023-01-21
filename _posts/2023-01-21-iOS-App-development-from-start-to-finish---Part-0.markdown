---
layout: post
title:  iOS App development from start to finish - Part 0
date:   2023-01-21 09:27:00 +0000
categories: [Blog,iOS,Swift]
tags:   [Blog,iOS,Swift]
image: assets/images/geile_eier.jpg
description: ''
featured: false
hidden: false
---
When I first started working on iOS apps it felt very arduous to actually find the right resources to learn. The internet is full of tutorials on setting up an Xcode project, how to use certain APIs, how to choose the right architecture or Appstore deployment. But there are not as many on the overall process. From choosing the right setup over the implementation to the final deployment. Basically everything and anything that one needs to learn to bring their app idea to the public. This is going to be a multipart series on exactly that. 

The target audience is supposed to be everyone that is interested in developing iOS apps using the Swift programming language. People that are more advanced in this profession will hopefully find something useful as well as those who are just getting started.

## **🦦 Structure**
All of the points mentioned below are going to be covered in one or more separate posts. Especially topics like system design and implementation will take bit more to comprehensively explain. The following topics will be presented in the given order.

> 🛠 Requirements and project overview 
>
> 📝 Sketching it out 
>
> ⛓ A little bit of system design 
>
> 👩‍💻 Implementation 
>
> 🌤 Putting it out there

The uncomfortable truth when working alone on an app project is that there is no one else that will take away parts of the job. Like designs, backend development, prioritisation or CI setup. But I feel it's good to do those things at least once by yourself to better appreciate the importance other positions.

## **🚀 Let's begin**
There is really not much left to do but diving into the process and making some magic happen. The first part of the before mentioned structure is actually quite small and straightforward.

What kind of app are we going build? Sorry if it's going to seem a little lame but the project is going to be a *flash card*app. The functionality is scoped down quite a bit, the user can create different stacks that can hold cards. Cards have a question and answer side. Stacks can be viewed in an exercise mode that will track the progress.

# **🛠  What you need**
The list of requirements is very manageable:

- A device that runs at least MacOS 12.X
*(MacBook, iMac, MacMini, ...)*
- Xcode >=13.X 
*(You can download it directly from the MacOS App Store)*
- Basic knowledge of the Swift programming language
*(All you need to know can be learned *[*here*](https://docs.swift.org/swift-book/)*)*
- An Apple Development account*(
To deploy the app to the App Store you are going to need a paid membership, for the development a normal one is just fine)*
If all of this is taken care of, you can start this journey.

## **🎬 Final Remark**
I know that it's going to be very hard not to jump into writing code directly but bear with me. Those first steps are really important, trust me.

![Alt](https://c.tenor.com/UsUDqY5lp0kAAAAC/mr-bean-wink.gif)
