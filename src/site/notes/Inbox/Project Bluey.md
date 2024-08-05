---
{"dg-publish":true,"dg-path":"Tinkering/project_bluey","permalink":"/tinkering/project-bluey/","tags":["📝/🌱️"],"noteIcon":"fern","created":"2024-07-30 21:42","updated":"2024-08-05 22:04"}
---

> [!warning] Current state
> This project is not finished jet. I will update the page on a regular basis

Recently, I watched [JvPeek's stream on Twitch](https://www.twitch.tv/jvpeek?lang=de). During the stream, I noticed that when viewers donate 300 Bits, there's a chance to trigger a soap bubble machine, which is controlled by an ESP. Inspired by this, I decided to build my own version of this device. To get started, I ordered the following parts:

- [Bluey Bubble Machine](https://www.action.com/de-de/p/3013160/seifenblasenmaschine/)
- ~~[USB-C Connectors](https://amzn.to/3ynJVhV)~~ → They did not work
- [USB-C Powermodule](https://amzn.to/4drznx0)
- [L298N](https://amzn.to/4c5shgv)
- [D1 Mini](https://amzn.to/3LLueEf)

## The idea

Inside the machine, there is a lot of space for my planned additions.
![IMG-20240804224523048.png|center|300](/img/user/Media/Inbox/Project%20Bluey/IMG-20240804224523048.png)

My plan is to add a D1 Mini with an L298N board to control the motor. Additionally, I want to be able to control the speed. Later on, I might add some LEDs to its eyebrows, which should be doable.

After a bit of planning, I decided to remove the inlay of the battery container to fit my own electronics there and maintain them if needed. This way, I only have to remove one screw to access my electronics.

## Building the "Upgrade"

### Adding the components
I placed one of the USB-C connectors in the back of the figure.
![IMG-20240804224523521.png|center|300](/img/user/Media/Inbox/Project%20Bluey/IMG-20240804224523521.png)
After experimenting for a while, I discovered that the USB-C connector lacked a control circuit inside. When I requested power, it essentially stopped working. So, I ordered a new board from Amazon and cut the cables on this one.

After fixing the power issue, I managed to fit everything inside the space behind the battery case. I placed some cardboard inside the case and used hot glue to secure the components in place.

![IMG-20240805220430077.jpg|center|300](/img/user/Media/Inbox/Project%20Bluey/IMG-20240805220430077.jpg)

### ESPHome Code
To control the motor, I use ESPHome, which is integrated with Home Assistant. This allows me to use automations and events from within my home to trigger the machine.

I use the built-in [fan component](https://esphome.io/components/fan/) to control the speed in Home Assistant.

```yaml
esphome:
  name: bluey
  friendly_name: Bluey

esp8266:
  board: d1_mini

# 
# Removed standard stuff here
# 

# Define the pins
output:
  - platform: esp8266_pwm
    id: motor1_forward_pin
    pin: D2
  - platform: esp8266_pwm
    id: motor1_reverse_pin
    pin: D1

# Use the fan component
fan:
 - platform: hbridge
   id: motor_1
   name: 'Blasebalg'
   pin_a: motor1_forward_pin
   pin_b: motor1_reverse_pin
   decay_mode: slow
   on_turn_on:
    - logger.log:
       format: 'Motor 1: on - duration = %d ms, direction = %d'
       args: ['id(motor_1_duration)', id(motor_1).direction]
   on_turn_off:
    - logger.log: 'Motor 1: off
```