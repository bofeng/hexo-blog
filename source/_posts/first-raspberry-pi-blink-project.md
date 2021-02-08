---
title: 'First Raspberry Pi Blink Project'
date: 2019-03-22 14:56:20
tags: [raspi]
published: true
hideInList: false
feature: 
---
Circuit:
<!-- more -->


![](/post-images/1571684199166.png)

Code:

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setup(18, GPIO.OUT)
for i in range(10):
    GPIO.output(18, GPIO.HIGH)
    time.sleep(1)
    GPIO.output(18, GPIO.LOW)
    time.sleep(1)

GPIO.cleanup()

```
