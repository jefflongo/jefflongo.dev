---
title: A Hockey Fan's Birthday Gift
categories: [Projects, Software, Hardware]
tags: ["python", "raspberry-pi"]
image:
  path: /assets/img/nhl_goal_siren/externals.jpg
  width: 1000
  height: 400
  alt: Siren and supporting electronics
date: 2025-02-28 18:04 -0800
---

## A Better Version of Budweiser's Product

I was perusing around Instagram when I came across Budweiser's "Red Light", which is a WiFi-enabled siren which activates when your favorite hockey team scores a goal. My uncle is a huge LA Kings fan, so this was a perfect gift for his upcoming birthday. This was until I found out they're selling it for $160! I thought to myself, surely I could make a better product for a heck of a lot cheaper. The Budweiser model is of dubious quality, requires batteries, and does not support some features which I would consider essential such as a programmable delay to sync it to your TV. So I was off to Amazon to purchase some parts.

## Hardware

The hardware for this project is rather simple. It consists of:

- A siren
- A relay to control the siren
- A Raspberry Pi Zero 2 W to run the software
- An Adafruit 3W speaker and I2S amp to play audio
- A simple hand-made LED/button UI for setting the delay

The hardware costed around $60. 

!["Hardware internals"](/assets/img/nhl_goal_siren/internals.jpg)
_Hardware internals_

The hardware is mostly plug and play. When the system is powered, it will light the siren and play a sound effect when a pre-programmed team scores. The only user interface is a button for cycling between four pre-programmed delay settings. There are three LEDs to indicate which delay setting is currently active.

The enclosure was designed and 3D printed in three parts: a center, top, and bottom. It was designed this way such that the electronics could be mounted on the top side, while the relay (which is switching 120VAC) could be mounted to the back side to isolate the high voltage from the rest of the system in the event of a mechanical failure. This also allocated room to include strain relief for the siren's power cable. I wanted to avoid switching 120VAC, but the siren I chose to use was switched on and off with a mechanical switch that connected the severed 120VAC cable, so switching this already severed cable was the cleanest way for this particular siren.

!["Relay mounting"](/assets/img/nhl_goal_siren/relay.jpg)
_Relay mounting and strain relief solution_

## Software

The software was written in Python, and is available on [GitHub](https://github.com/jefflongo/nhl-goal-siren). It primarily relies on [NHL-api-py](https://github.com/coreyjs/nhl-api-py), which is a Python package that provides a wrapper around the NHL API. The NHL API can be used to get all sorts of detailed information, such as stats, rosters, standings, schedules, and live game updates. For my use case, I only cared for a small subset of the functionality: game schedules, live scores, and whether a game is currently active or not. 

The program's state machine is either waiting for a game to start, or actively monitoring a live game. When waiting for a game to start, it queries the API for the current week's schedule of games containing the team of interest. This query is performed once an hour, tracking the nearest upcoming game. Once the game starts, the switch is made to monitoring mode. In monitoring mode, the API is queried for the current score once a second (there is no way to register a callback on a team score). If the team of interest's score changes from the last query, the siren and sound effect are activated after the selected delay period. Once the API reports that the game is no longer live, the program returns to "waiting for a game" mode.

The user interface and relay are controlled using the `rpi-lgpio` Python package, which made it simple to control GPIO pins, register an interrupt for the button, and even provide software debouncing for the button.

The software contains some other features, such as a command line interface for choosing a team to monitor, and a configuration file for persisting the current delay setting. The program is set up as a systemd service, which causes the program to be executed as soon as the Pi turns on. Furthermore, if the program crashes due to a bug or error, systemd will automatically restart it.

## The Finished Product

The end result is exactly what I was hoping for. The only real difficulties I encountered was selecting appropriate delays from empirical testing. I had some trouble getting a large enough sample size, because every LA Kings game I watched resulted in them getting destroyed and not scoring!

{% include embed/video.html src='/assets/img/nhl_goal_siren/goal.mp4' %}

If I had more time to work on this project, I would likely develop a phone app to go along with the hardware. The Raspberry Pi could be configured as a Bluetooth peripheral in the Python application pretty easily with BlueZ. This would allow for a much more detailed configuration, such as configuring WiFi networks without pulling the SD card, dynamically changing the team of interest, programming exact delays, and volume adjustment. Another option would be to add a touch screen and a user interface that way, which I omitted due to time constraints. Despite this, the project was, overall, a huge success.
