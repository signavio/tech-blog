---
title: "Code Camp Projects 2017"
description: ""
tags: code camp
author: Rachel Stiles
layout: article
image:
  feature: /2017/codecamp.jpg
  alt: ""
---

Every year, Signavio's hard-working engineering department takes a break from their pursuit of BPM excellence to participate in Code Camp, a yearly hackathon.
They divide into groups, brainstorm ideas, and get to work creating a variety of projects.
At the end of Code Camp, they present their finished projects to the entire company.
The best project is selected by vote via the company, and it becomes the newest project that our team works on.

This year, we had 10 groups working on a variety of projects, from a Slackbot integration with our Workflow Accelerator to a re-design of our Editor's shape menu. Here is a quick summary of each.

**Note** These are simply projects our engineers worked on for challenge and fun--they may never actually make it into production.


##A better Shape Menu

When the Editor first launched, the menu was revolutionary. It was easy to use, it allowed the user to see all tool options in a clear way, and it was a pleasure to navigate.
As with all good ideas, the design was eventually adopted industry-wide, and is now not so remarkable.
This group looked at the menu and though--how can we improve this? How can we create a new menu that is every bit as intuitive and astounding as the original version was when it first released?

The solution they came up with includes bigger icons, a less crammed menu, and adding a touch function.
The menu this group designed also has two modes--the quick mode, which has bigger icons and a more user-friendly menu designed with touch screens in mind; and what they call the rocket mode, which text-based and designed with modeling speed in mind.
According to the group, using the rocket mode can cut modeling time in half.

## Mix Me

At Signavio, we have a monthly tradition called Mix & Match Lunch where we randomly divide into groups of people we don't usually interact with, and go to lunch.
It's a way to ensure we know everyone we're working with, even if we don't see them every day.
This app is a way to extend that tradition to the entire week, maintaining our company culture even as we grow.

Mix Me is a progressive web app, utilizing REST and Note.js.
A user logs onto the Mix Me webpage, enters their Signavio email and is randomly paired up with a group of four other colleagues, a random restaurant near Signavio HQ and a suggested meeting place.

## SWAccelerator Talk - Alexa + Workflow Accelerator

This project focused on an actual use-case that involves integrating Alexa with our Workflow Accelerator, utilizing Note.js.

Say that a company's social media team wants to allow employees to send out tweets, but is unsure how to run a quality control process that would prevent any gaffes that might embarrass the company.
Using Workflow Accelerator, you would be able to integrate it with Alexa.
Simply dictate the tweet to Alexa, who will compare the text of the tweet to its library of acceptable syntax and phrasing.

If the tweet is approved, the Workflow Accelerator will send an email to the social media team alerting them to the tweet and allowing them to either approve it or reject it.
If Alexa determines the tweet is not acceptable, it will reject it and ask the user to try again.

While Alexa serves as a filter for content, the ultimate authority that approves or disapproves tweets is the company's social media team.

## Fast. Frictionless. Fun.

Ever wished Signavio had an app version?
Not to create diagrams with, but an app that would allow you to see the core information of your projects, write down notes, send feedback, etc. all while you're on the go?
Well, so did we!

This group focused on making a mobile version of our Workflow Editor for Android.
The main challenge for this group was making things lighter and faster than the desktop version, with comments and share capabilities as well as a way to check in on project details while on the go.

The resulting app is a mobile version of our Workflow Editor, minus the workflow capability.
This group focused on making the app as light and fast as possible, ensure it is optimized for mobile use.
Users can comment, share snapshots of process diagram models, make notes and similar tasks, which makes this mobile app a sort of one stop shop for info about any process models you may be creating.

Mobile app--dashboard for comments, page views, menu for people to browse folders for the editor
Mobile editor with feed for comments and a snapshot of the models--but not the actual models themselves--just like a one stop shop for info about it
Android for now

## Quiet-o-matic 5000

Like many tech companies, Signavio has an open-plan office.
While we do our best to be respectful of each other, sometimes things can get a little loud.

The solution our engineers came up with is a traffic light that alerts the room when things are getting a little loud.
Using Raspberry Pi and a microphone, hooked up to a physical light and using a program our devs wrote to make it all talk to each other.
The program is set to detect a certain volume level, and to announce (via a green, yellow or red light) when changes in that level are detected.

Helpfully, the app also tracks and graphs noise levels over time, allowing the admin to see what the noisiest times of the day are and what causes certain spikes in noise.

## Slacknavio

Sometimes, when using Signavio, a user wants to check something small--conversations they're a part of, perhaps, or maybe just double-check the tasks they are assigned--but they don't want to have to log into Workflow Accelerator or the Collaboration Hub just for a quick look.

That's where the Slacknavio bot comes in.
This is a Slack bot that allows users to pull basic information from Workflow Accelerator or the Collaboration Hub instantly.

Created with [Go] (https://golang.org/), the Slacknavio bot can post updates about new diagrams in the Collaboration Hub, new personal tasks assigned to users in the Accelerator, notify users of new comments in conversation threads they are included in, as well as search the Signavio repository and share daily status updates on things such as workflow processes.

## Workflow4Slack

Let's say you want to more easily track what is going on in the Workflow Accelerator, but you don't want to wait for your colleagues to get the email notifications.
This project integrates Slack with the Workflow Accelerator to notify users when changes have been made in their workspace.

Want to notify a user that they've been assigned a new task? Workflow4Slack will ping them.
Made changes to a process and need some feedback from your colleagues?
You can send them a notification from Workflow Accelerator via Slack.
It's a good option for those times when email simply takes too long.

## Oops, did I do that?
When a bug is discovered, it can sometimes be a challenge to locate where exactly it is in the code.
When you're collaborating with many people on a team, it can also be a bit tricky to find a solution, because it's not always easy to figure out how the bug happened or what caused it or if it was caused by a change someone else made.

This program seeks to make the de-bugging process a little easier by creating a log.
Users can register bugs as they find them, along with the location of the bug, as well as find out who last worked on that part of the code.


##Confluence BPMN Diagram Plugin

A plugin that allows the user to integrate Confluence with our Signavio suite

##Gmail Signature Updates

Keeping the company email signatures up to date is always a challenge, especially when the company is growing as quickly as Signavio.
This group came up with a simple solution-create a centrally managed program that works with Google Drive to make it simple to update email signatures.
