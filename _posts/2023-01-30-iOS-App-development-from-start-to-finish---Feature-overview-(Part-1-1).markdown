---
layout: post
title: iOS App development from start to finish - Feature overview (Part 1-1)
date: 2023-01-30 10:40:00 +0000
author: raoul
categories: [Blog,iOS,Swift]
tags: []
image: assets/images/paula_tongue.png
description: "📝 Sketching it out."
featured: false
hidden: false
---
If you have not read the first part of the series I'd highly recommend doing so as it gives an introduction on what this is all about: [Part 0](https://raoul-schwagmeier.ghost.io/tech/app-development-from-start-to-finish-part-1/)


Again, those first few parts are going to be a bit of a drag to read as they are purely about the what and how. Building a piece of software can be compared to baking a cake. It usually is a lot easier when you know what tools to use and what ingredients to put in before you start. This part is basically about describing what kind of cake we are baking or in other words what app we are programming and the ingredients that it's made from.

![Alt](https://c.tenor.com/5SKA5XuifgoAAAAC/swedish-chef-baking.gif)
Alright, enough with the analogy. The project is a flashcard learning app.

## **🌍 Feature overview**
I'm for sure not an expert in project planning but what I usually do is laying out the core functionality and work my way around that. The most important features are:

- A way of organising the cards: I will call them **Decks**
- Editing decks by adding, removing or rephrasing cards
- Viewing the cards in a deck
- Learning with decks: There is going to be a whole section about this part


Having those will make up a pretty ok-ish learning app. But *"pretty ok-ish"* is not what I want and probably neither most users. To improve on the concept the app should have some additional features:

- Notifications at fixed hours as learn reminders
- Progress tracking


As I have, at this point in time, no idea what the final app is going to look like, this list ist most likely to change. It's important to keep that in mind as in real life requirements can change on a daily basis. But having at least an idea of what there should be helps a lot in maintaining focus on the result.

## **🌊 App flow**
Before actually "sketching it out" I want to define the app flow as a measure to put the before mentioned features into a context. The following image gives a very simplistic view on how everything is connected.

![Alt](https://github.com/Arestronaut/arestronaut.github.io/blob/main/assets/images/app%20development/AppFlow.png?raw=true)


Because this was not mentioned before, the onboarding is used to setup some personalisations. It will only be displayed when the users open the app for the first time. Otherwise they will be navigated to "Decks Overview" which will act as the app's main screen.

## **👋 Outlook**
![Alt](https://c.tenor.com/UNgMDLiHVZ0AAAAC/annie-edison-annie.gif)


Wowee! Still about 98% work left but it's like learning to walk for the first time: one wobbly step at a time. Now that it's more clear what we are going to build we can creating a design concept. With a color palette, custom font, screens and maybe I'm being fancy: some animations. But this will be done in the next part 💚

