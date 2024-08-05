---
{"dg-publish":true,"dg-path":"Tinkering/project_bluey","permalink":"/tinkering/project-bluey/","tags":["📝/🌱️"],"noteIcon":"fern","created":"2024-07-30 21:42","updated":"2024-08-05 20:53"}
---

> [!warning] Current state
> This project is not finished jet. I will update the page on a regular basis

Recently I did watch [JvPeeks](https://www.twitch.tv/jvpeek?lang=de) Stream on Twitch. If you donate 300 Bits to him, there is a chance to release some soap bubbles. For this he uses a soap bubble machine which is controlled by an ESP. So I decided to built my own version of such a litte device. I ordered/bought the following parts:

- [Bluey Bubble Machine](https://www.action.com/de-de/p/3013160/seifenblasenmaschine/)
- [USB-C Connectors](https://amzn.to/3ynJVhV)
- [L298N](https://amzn.to/4c5shgv)
- [D1 Mini](https://amzn.to/3LLueEf)

## The idea

Inside there is a lot of space to add my planned additions.
![IMG-20240804224523048.png|center|300](/img/user/Media/Inbox/Project%20Bluey/IMG-20240804224523048.png)

My plan is to add the D1 Mini with the L298N-Board to control the motor and in addition I want to be able to control the speed. After that I may add some LEDs inside its eyebrows - that should be doable.

After a bit of planning I did decide to remove the inlay of the battery container to be able to fit my own electronics there and maintain them if needed. So I only have to remove one screw to be able to reach my electronics.

## Building the "Upgrade"

### Adding the components
I placed one of the USB-C connectors in the back of the figure.
![IMG-20240804224523521.png|center|300](/img/user/Media/Inbox/Project%20Bluey/IMG-20240804224523521.png)
After trying around for a while I discovered, that the USB-Connector had no control circuit inside. So when I did request some power it basically stopped working. I ordered a new board on Amazon and cut the cables on this one …

After fixing the power issue I manged to fit everything inside the space behind the battery case. I simply placed some cardboard inside the case and used hot glue to keep the components in place.

### ESPHome Code
To control the motor I use ESPHome which is part of Home Assistant. So I am able to use automation's and events from inside my home to trigger the machine.

I use the builtin [fan component](https://esphome.io/components/fan/) to be able to control the speed in Home Assistant.

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

## Problems
### Power connection issue
Currently, I have the following problem: When I connect the ESP and the Power separate everything works fine. The moment I connect both to the same input, everything stops working. I think it has something to do with the connector I am using.