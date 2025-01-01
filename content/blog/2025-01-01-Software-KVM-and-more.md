<!-- title: Software KVM and other home controls with MQTT -->
<!-- description: Creating a MQTT based automation tool for use with Home Assistant. -->
I've been building my Home Assistant setup for about a year now, adding more integrations, connecting to my various smart devices, and building the dashboard. Last spring I bought a [Waveshare 10" touch display](https://www.waveshare.com/wiki/10.4HP-CAPQLED) and asked a colleague to print me some legs I found a design file for. This gave me a nice dashboard for my desk.

![Home office desk HA dashboard](/images/2025/ha-dashboard-touch-panel3.png)

The next step was taking advantage of my Dell Ultrasharp's built-in KVM to switch between my work and personal computers without reaching behind to the controls. I know, it has built-in hotkeys for this functionality, but I'm not thrilled by the idea of something intercepting keystrokes and why go simple when you can over engineer.

## Software monitor control

For those who have never heard of DDC/CI (Display Data Channel Command Interface) it implements a command protocol over I2C which most modern monitors support. You can send commands like change brightness, power off, and.... switch inputs. There's a package called [ddcutil](https://www.ddcutil.com/), and since my personal computers are all linux - an obvious synergy emerged, I could send commands to switch inputs, somehow.

## MQTT

One of the great functionalities of HA is the MQTT integration it has, the ability to easily send messages. This felt like the perfect method to send commands to a remote listener on the desktop to issue these commands.

```json
[
  {
    "command": "monitor",
    "data":
      {
      "monitor": "left",
      "input": "hdmi",
      "power": "wake"
      }
  },
  {
    "command": "monitor",
    "data":
      {
        "monitor": "right",
        "input": "usbc"
      }
  },
  {
    "command": "speaker",
    "data":
      {
        "volume": "up"
      }
  },
  {
    "command": "lock",
    "data":
      {
        "unlock": true
      }
  }
]
```

## Golang and Switcher

When I started this project learning Golang had been on my radar for a while, so I forced myself to use it. Python might have been simpler for my existing knowledge but actually with the help of Copilot it took less than a day to get a working prototype functioning simply called [Switcher](https://github.com/lairdm/switcher).

The more challenging piece was getting it running the way I wanted, I have dockerize about a dozen different services on this machine and wanted to do the same with `Switcher`. Unfortunately after a lot of attempts and failures, I was able to map the proper I2C devices to the container but something just didn't quite work right and the application kept core dumping whenever it tried to run ddcutil. It wasn't worth the fight in this case so I went with Plan B, run it using Supervisor, it works.

But once working it was simple enough to add to my HA Dashboard. You can see these buttons in the bottom left of my desk display.

```yaml
- type: horizontal-stack
  cards:
    - type: custom:mushroom-template-card
      primary: Display
      secondary: ''
      icon: mdi:monitor
    - type: custom:mushroom-template-card
      primary: Laptop
      secondary: ''
      icon: mdi:laptop
      tap_action:
        action: call-service
        service: mqtt.publish
        service_data:
          topic: /home/office/monitors
          payload: >-
            [{"command": "monitor", "data":
            {"Input":"usbc","Monitor":"right"}}]
    - type: custom:mushroom-template-card
      primary: Desktop
      secondary: ''
      icon: mdi:desktop-tower
      tap_action:
        action: call-service
        service: mqtt.publish
        service_data:
          topic: /home/office/monitors
          payload: >-
            [{"command": "monitor", "data":
            {"Input":"hdmi","power":"wake","Monitor":"right"}},
			{"command":"lock","data":{"unlock":true}}]
```

## Enhancements

Now with this working I moved on to some enhancements. What about turning the display back on and unlocking the machine when switching. One of the slowest parts was after pushing the dashboard button the monitor had to switch all the USB inputs over, the computer has to notice the mouse again, then I had to wiggle it to turn off screen blanking. By using `xset` to wake everything up it cut the switch time in half in cases when the desktop had blanked the screen.

As well, my long term plan is a headless Plexamp instance on a Pi which can be centrally controlled. This way when switching back and forth I'd have the same playlist going. This needed the ability change the volume on the host machine as well. So commands for `amixer` for volume which `Switcher` could handle too seemed obvious.

The final enhancement to come is switching the input of my Edifier speakers automatically. I've purchased some <b>KY-005 Ir Emitters</b> and found a [fantastic write-up](https://marabesi.com/iot/mapping-ir-emitter-k005-receiver-ky-022-to-raspberry.html) on how to use these with `lirc` on a Pi. I'll publish another article when I get that working.
