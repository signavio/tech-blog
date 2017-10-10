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

Every year, Signavio's hard-working engineering department takes a break from their pursuit of BPM excellence to participate in Code Camp, a yearly hackathon.
They divide into groups, brainstorm ideas, and get to work creating a variety of projects.
At the end of Code Camp, they present their finished projects to the entire company.
The best project is selected by vote via the company, and it becomes the newest project that our team works on.

This year, we had 10 groups working on a variety of projects, from a Slackbot integration with our Workflow Accelerator to a re-design of our Editor's shape menu. Here is a quick summary of each.

Note that these are simply projects our engineers worked on for fun--they may never actually make it into production.

##A better Shape Menu
Quick mode: bigger icons, less crammed menu, touch friendly
Rocket mode: Text box based--the process will predict what model you want to create. Never have to use the mouse, only the keyboard.
Modeling time cut in half

When our Workflow Editor first launched, the Shape menu was revolutionary.
It was clear, easy to use and easy to access.
It was such a good design, in fact, it was quickly adopted by our competitors.
However, improvements can always be made.
The team for this project set out to re-design the Shape menu to make it better fit today's users.
This included bigger icons, a less crammed menu, and adding a touch function.
The menu this group designed also has two modes--the quick mode, which 




## Mix Me

At Signavio, we have a monthly tradition called Mix & Match Lunch where we randomly get into groups of people we don't usually interact with, and go to lunch.
It's a way to ensure we know everyone we're working with, even if we don't see them every day.
This app is a way to extend that tradition to the entire week, maintaining our company culture even as we grow.

Mix Me is a progressive web app, utilizing Rest.js and Note.js.
A user logs onto the Mix Me webpage, enters their Signavio email and is randomly paired up with a group of four other colleagues, a random restaurant near Signavio HQ and a suggested meeting place.


## SWAccelerator Talk - Alexa + Workflow Accelerator

Note.js, other processes, Alexa

This project focused on an actual use-case that involves integrating Alexa with our Workflow Accelerator.
Say that a company's social media team wants to allow employees to send out tweets, but is unsure how to run a quality control process that would prevent any gaffes that might embarrass the company.
Using Workflow Accelerator, you would be able to integrate it with Alexa.
Simply dictate the tweet to Alexa, who will compare the text of the tweet to its library of acceptable syntax and phrasing.
If the tweet is approved, the Workflow Accelerator will send an email to the social media team alerting them to the tweet and allowing them to either approve it or reject it.

If Alexa determines the tweet is not acceptable, it will reject it and ask the user to try again.

While Alexa serves as a filter for content, the ultimate authority that approves or disapproves tweets is the social media team.


The point of this project is to show the potential of our Workflow Accelerator when it comes to orchestrating different services and processing inputs with a set of other services (ie, Alexa).

It will allow all the different stakeholders to offer their input, and then hand it over to the developers to code a solution that will use the Workflow Accelerator as a process interface.  
The chief concern is an easy to use integration that does not require a lot of coding to customize to a customer's needs.  


## Quiet-o-matic 5000

The idea behind this project is to manage the noise levels in an open-plan office. Using Raspberry Pi and a microphone, hooked up to a physical light and using a program our devs wrote to make it all talk to each other.
The program is set to detect a certain volume level, and to announce (via a green, yellow or red light) when changes in that level are detected.
Users can also (anonymously) go into the app and manually operate the light when they feel the noise levels are too much for them to concentrate.

## Workflow4Slack

This project integrates Slack with our Workflow Accelerator.

## Oops, did I do that?

Register bugs, see who last worked on the code, find out where the bugs are in the code

Uses Git, slava.jack

##Confluence BPMN Diagram Plugin

A plugin that allows the user to integrate Confluence with our Signavio suite

##Gmail Signature Updates

A centrally managed program that uses Google Drive as a way to keep our company signatures updated with minor input from users

##A better Shape Menu
Quick mode: bigger icons, less crammed menu, touch friendly
Rocket mode: Text box based--the process will predict what model you want to create. Never have to use the mouse, only the keyboard.
Modeling time cut in half


## Slacknavio

This is a bot that will allow users to pull information from Workflow Accelerator basically instantly. The information will include [x] [y] [z]

To use it, the employee will open up Slack at work and summon the bot into the channel where the information needs to be shared. It is programmed using [x] which was difficult to use.

## Fast. Frictionless. Fun.

This team had a simple idea: let’s make a mobile app that allows users to see the core information of their process management projects while  they’re on the go.
Maybe you’re in a meeting and need to quickly reference the basic facts of your case. Maybe you’re working remotely and forgot some details.
Either way, this app will help you with giving you what you need.

This group focused on making a mobile version of our Workflow Editor for Android.
The main challenge for this group was making things lighter and faster than the desktop version, with comments and share capabilities as well as a way to check in on project details while on the go.

Mobile app--dashboard for comments, page views, menu for people to browse folders for the editor
Mobile editor with feed for comments and a snapshot of the models--but not the actual models themselves--just like a one stop shop for info about it
Android for now
A mobile version of the editor, minus the workflow capability
Making it lighter and faster than the desktop version
Search, comment, etc.
Better user interface, optimized for mobile



## Editor Shape Menu re-design

When the Editor first launched, the menu was revolutionary. It was easy to use, it allowed the user to see all tool options in a clear way, and it was a pleasure to navigate.
As with all good ideas, the design was eventually adopted industry-wide, and is now not so remarkable.
This group looked at the menu and though--how can we improve this? How can we create a new menu that is every bit as intuitive and astounding as the original version was when it first released?

To achieve this, they thought of a variety of solutions. Gesture-based, click based, anything that would lessen the amount of work and clicks that a user currently needs to use the Editor.
