---
layout: post
title: iOS App development from start to finish - UI design (Part 1-2)
date: 2023-02-14 07:32:00 +0000
author: raoul
categories: [Blog,iOS,Swift]
tags: []
image: assets/images/art_gallery.jpg
description: ""
hidden: true
---
Just as a huge disclaimer: I have no real clue about UI design. Of course when working with with designs all day long you get an idea of what a good design might look like. I did my best to create something that works well and is aesthetically pleasing at the same time.

## **What I worked with**
The main tool I used for creating a design is [Figma](https://www.figma.com/). If you don't have multiple projects at the same time it's free to use in mostly full capacity. One of the more helpful things about Figma is the massive community that creates free to use resources. For example I heavily used [SF Symbols](https://www.figma.com/community/file/886999666531731323) and the [Emoji Pack by Paca](https://www.figma.com/community/file/937774188065101204). With those two a vast abundance of resource is at your hand. Icons and fun images to use throughout the design. For coming up with a colour scheme I used [Open Color](https://yeun.github.io/open-color/#green) a set of colours optimised for UI design.



Figma is a quite easy to use tool and google is a big help in accomplishing what you have in mind.



> Small Intermezzo: Before I can actually showcase something I first need to present the final name of the project because it will set a general theme. It's **Learnephant** (Learn + Elephant)



## **The design**
Without talking too much around the hot porridge (a german saying) let's take a look a the design.



### **Color scheme**
As a color scheme I nailed it down to the following:

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/color-scheme.png?raw=true)


## Onboarding
The app flow begins with an onboarding that will give a short introduction in the app and will set up the necessary permissions

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/onboarding.png?raw=true)


All of those information are optional so the user could theoretical just click through the screens without providing any information. Everything can be setup later in the preferences. The small elephant at the bottom acts as a fun progress indicator.

## Main page
After finishing the onboarding (resp. as the main entry point) comes the main page.

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/main.png?raw=true)


The main function of this screen is to display all the decks and of course as a natural consequence also creating new decks. The deck cells can be interacted with in two ways:

- By long pressing a context menu will appear that can be used to delete or edit the deck
- By clicking on the cell the user will be navigated to the deck's detail


The upper right button will navigate the user to the preferences screen.

## **Deck Detail**
The deck detail screen will hold the a lot of information. In order to make it less hectic it's divided into the "Dashboard" and the "Cards Overview".

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/deck-detail.png?raw=true)


There is still a lot going on, let's break it down:



The dashboard will hold statistics about the deck. At this point I don't have the full picture on how data will look like therefore this screen is likely to expand. At the top the user will be able to the their current progress and can start a learning session.



In the cards overview the user can create new cards, edit or remove existing ones. When clicking the "create" button on the bottom sheet the card will be created and the form cleared but not closed. That is done to enable quick creation of new cards. When clicking on an existing card the same bottom sheet will be opened to edit the card's information.

## Learning
The "Start learning" button will start initialise the learning flow.

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/learning.png?raw=true)


How it works is the following: A learning session will be started with a given set of cards. The card selection will get it's own dedicated post. By clicking on a card it will present it's answer resp. the question again. When presenting the answer side of a card there are "Wrong"/"Correct" buttons that are used to choose whether the user has answered the question correctly. After the last question has been answered a screen to summarise the learning session will be presented.

## **Conclusion**
Of course not every screen is completely laid out and the existing ones are also very likely to change during development. That's just how it is. Not even the best developer or designer can foresee all the possible scenarios of a fully working application. But having at least a few screens, a color scheme and a general idea on screen flow will act as general framework that is used to build the final product.



The next posts will be about system design: Defining the architecture, identifying different key components and bring it all together. These parts will be a bit more close to actual development.



![Alt](https://media.tenor.com/_0SyvC_r2XAAAAAC/hi-hello.gif)
