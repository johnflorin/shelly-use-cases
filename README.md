# shelly-use-cases
Automations that make life easier with Shelly &amp; Home Assistant


Home Assistant is an excellent smart home platform and, combined with Shelly products, one can have automations with no cloud involvement at all (but it is optional, if needed).

My project includes these scripts & automations (as defined in Home Assistant's scripts.yaml & automations.yaml) with Shelly products:


1. Bedtime for Nina - if it's 20:30 and my kid's Shelly-controlled (with a Shelly 2.5 relay) window roller shutter is still open, Google Home speakers throughout the apartment tell her to go to sleep: 

```

- alias: Bedtime for Nina
  trigger:
  - platform: time
    at: '20:30:00'
  condition:
  - condition: state
    entity_id: cover.shellyswitch25_e8db84a1e866
    state: open
  action:
  - service: media_player.volume_set
    data:
      volume_level: 0.5
  - service: tts.cloud_say
    data:
      message: ' Nina, it's time to go to sleep. '
      entity_id: media_player.mismartclock0741,media_player.googlehomehub2498,media_player.master_bedroom_pair_2,media_player.googlehome3247,media_player.bedroom_speaker
  initial_state: 'on'
  mode: single

```

2. The dryer is finished, so it should be emptied - I used a Shelly Plug S to do energy usage fingerprinting (via Grafana, pictured below) to identify the dryer's full cycle and used a Home Assistant blueprint to send me a Telegram notification when the dryer is done:

![image](https://user-images.githubusercontent.com/45756085/139138463-a54fa35c-8666-4457-a02a-3819f4bf4dbc.png)


```

- alias: Dryer has finished
  description: ''
  use_blueprint:
    path: Sbyx/notify-or-do-something-when-an-appliance-like-a-dishwasher-or-washing-machine-finishes.yaml
    input:
      power_sensor: sensor.shellyplug_s_c16e05_power
      starting_threshold: 100
      starting_hysteresis: 2
      finishing_threshold: 10
      finishing_hysteresis: 10
      actions:
      - service: notify.telegram_florin
        data:
          message: The dryer has finished, please empty it.
          title: Dryer Finished

```

3. In order to save power, I turn my PC off at night via an RPC soft shutdown command & a Shelly Plug S then cuts power completely to the power strip that includes the PC, its monitor and other dumb (but power-consuming) devices which are not used at night. I also close the living room window roller shutter (controlled with a Shelly 2.5 relay) via the same bedtime script:

```

bedtime:
  alias: Bedtime
  sequence:
  - service: cover.close_cover
    target:
      entity_id:
      - cover.shellyswitch25_3c6105e5b15a
  - service: hassio.addon_stdin
    data:
      addon: core_rpc_shutdown
      input: Yngvi
  - delay:
      hours: 0
      minutes: 2
      seconds: 0
      milliseconds: 0
  - type: turn_off
    entity_id: switch.shellyplug_s_c18ff9
    domain: switch 

```
