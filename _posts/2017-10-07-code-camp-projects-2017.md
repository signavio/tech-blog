---
title: "Code Camp Projects 2017"
description: ""
tags: code camp, engineering
author: Rachel Stiles
layout: article
image:
  feature:
  alt:
---

Every year, Signavio's hard-working engineering department splits into groups to work on a variety of projects that they create from brainstorming. They present these projects at the end of the yearly Code Camp in front of the entire company. The best project is selected by vote via the company, and it becomes the newest project that our team works on.

This year, we had 10 groups working on a variety of projects, from a private, in-house Slack services to a re-design of our Editor's shape menu. Here is a quick summary of each.

###### Mix-Me Mobile Lunch App

This app is an expanded idea of the Wednesday weekly tradition.

Described as "Tinder for food", this is an internal app for employees of Signavio. When a Signavian signs up for the service, they are randomly paired with a group of colleagues based on what they feel like having for lunch that day. The user can swipe "yes" or "no" on the group they are paired with.


###### Noise Levels

This is the brainchild of Sascha, our head Agile coach. A traffic light is hooked up to a microphone and an app. How it works is that the microphone allows the program to detect how much noise is being created in the office. A default noise level is set. The microphone allows the app to determine if the noise being created is above the default noise level. It will display a green light if it’s okay, a yellow light if things are getting too noisy, or a red light if things need to be quieted immediately. In addition, a user can anonymously go into the app and trigger the light manually if they feel the office is getting too noisy for them to concentrate.

The idea, Sascha says, is to generally create an environment where people are aware of the noise they are creating and to try to be more considerate to their colleagues, regardless of where they are.

The physical light ended up not working, so our clever developers ended up creating a graphical interface with code. The program they wrote also allows users to see the graph of noise over time, so we can monitor how loud we really as a group and adjust accordingly.

###### Slacknavio


###### Fast Frictonless Fun

This group focused on making a mobile version of our Workflow Editor for Android. The main challenge for this group was making things lighter and faster than the desktop version, with comments and share capabilities as well as a way to check in on project details while on the go.


###### Alexa

This project focused on an actual user-case that involves integrating Alexa with our Workflow Accelerator. Say that a company's social media team wants to allow employees to send out tweets, but is unsure how to run a quality control process that would prevent any gaffes that might embarrass the company. Using Workflow Accelerator, you would be able to integrate it with Alexa. Simply dictate the tweet to Alexa, who will compare the text of the tweet to its library of acceptable syntax and phrasing. If the tweet is approved, the Workflow Accelerator will send an email to the social media team alerting them to the tweet and allowing them to either approve it or reject it.

If Alexa determines the tweet is not acceptable, it will reject it and ask the user to try again.



Particular solution, use-case--shows potential of accelerator when it comes to orchestrating different services and Alexa inputs and processing inputs with a set of other services
Shows power of workflow accelerator that makes this orchestration possible from a business perspective
Use it with all the different stakeholders, hand it over to developers
Note.js use to integrate with a few lines of codes
Actual use case: tweet approver
Say something like “Alexa, tweet” then dictate tweet
The system automatically runs a check--grammar, political-correctness, etc. (it compares it to a library via a process) (ie, “instead of using ‘cripple’ try ‘person with a limp’”)
Does not need a lot of coding
It gives you feedback
It will reject your tweet if your phrasing is bad
If approved, it is handed over to social media team, which can approve or reject
The tweeter will get feedback from Alexa, but social media does editing and approval--they get an email from Alexa
