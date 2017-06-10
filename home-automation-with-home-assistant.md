% Home Automation with Home Assistant
% ![](./media/email-icon.png){height=40 style=vertical-align:middle} jjmontes@gmail.com - ![](./media/github-icon.png){height=40 style=vertical-align:middle} jjmontesl
% PyDay Galicia 2017 (*CC BY-SA 3.0*)


# Home Automation

## Home Automation

Home Automation allows us to **control** \
and **automate** devices _at home_.

* Currently there is no standard for (home) IoT
* Different apps and clouds for each vendor
* Impossible to automate across boundaries

A Home Automation Hub connects to different devices and
manages events and commands across them.

## Applications

* Automate lighting and appliances based on presence, schedule, sun position...
* Wakeup alarms: lights, TV, music, blinds, coffee...
* Control heating using sensors, weather predictions, schedule or presence of family members.
* Alarm systems, presence detection, fake presence during holidays.
* Automate gardens using weather predictions and sensors (temperature, humidity...).
* Check if the car is low on fuel (set alarm 10 minutes earlier, send a notification).

# Home Assistant

## Home Assistant

![](./media/hass-icon.png){width=20%}

[https://home-assistant.io/](https://home-assistant.io/)

Home Assistant is an open-source home automation platform running on Python 3.
Track and control all devices at home and automate control. Perfect to run on a Raspberry Pi.

---

![](./media/hass-screenshot-01.png){width=100%}

---

![](./media/hass-screenshot-02.png){width=49%}
![](./media/hass-screenshot-03.png){width=49%}


## Devices

![](./media/hass-tech-featured.png){style=float:right;margin-top:100px;width:33%;}

<div style="width: 65%; float: right;">
[600+ ready to use components](https://home-assistant.io/components/):

* Lights, switches, covers, fans, sensors
* Thermostates, cameras, alarms
* Media Players, TVs
* DIY: Arduino, GPIO...
* Weather, ITTT, API.AI, Voice, TTS
* Track things on a map
</div>


# Example

## Installing

Needs Python3. Using PIP:

```
python3 -m venv env
. env/bin/activate
pip install homeassistant
```

## Configuration

* Configuration is by default in the `~/.homeassistant` directory.
* Configuration is defined in YAML format.
* Configuration starts in the `configuration.yaml` file.

_Tips_:

* Fixed IP addresses (configure your router or DHCP server)
* Names for hosts (prepare to edit /etc/hosts where needed)
* Avoid monolithic config file: use [homeassistant packages](https://home-assistant.io/docs/configuration/packages/)

##

configuration.yaml

<div style="font-size: 75%;">
```yaml
# Main config
homeassistant:

  # Location required to calculate the time the sun rises and sets
  latitude: !secret home_lat
  longitude: !secret home_lon
  # Impacts weather/sunrise data (altitude above sea level in meters)
  elevation: 75

  # Units: metric for Metric, imperial for Imperial
  unit_system: metric

  # Timezone (from: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
  time_zone: Europe/Madrid
```
</div>

---

<div style="font-size: 75%;">
```yaml
# Track the sun
sun:

# Enables the frontend
frontend:

# HTTP configuration
http:
  api_password: !secret api_password
  use_x_forwarded_for: True
  #base_url: !secret base_url
```
</div>

* component attributes are defined as a dictionary.
* `!secret` allows to define secrets in a separate file.

---

<div style="font-size: 75%;">
```yaml
# Zones
zone:

  - name: Home
    latitude: !secret home_lat
    longitude: !secret home_lon
    radius: 100
    icon: mdi:home

  - name: Work
    latitude: !secret work_lat
    longitude: !secret work_lon
    radius: 100
    icon: mdi:worker
```
</div>

Zones are used for device or user tracking.

## Running

Run homeassistant:

    . env/bin/activate
    hass -c configdir

Connect to:

[http://localhost:8123](http://localhost:8123)

## Devices: Sensors, Lights, Switches...

Add an item for each device to be controlled:

<div style="font-size: 75%;">
```yaml
switch:
  platform: command_line
  switches:
    kitchen_light:
      command_on: touch /tmp/kitchen_light
      command_off: rm /tmp/kitchen_light

sensor:
  - platform: command_line
    name: CPU Temp
    command: "/bin/cat /sys/class/thermal/thermal_zone1/temp"
    unit_of_measurement: "ºC"
    value_template: '{{ value | multiply(0.001) }}'
```
</div>

## Splitting configuration

You can split configuration per *platform* (switches, lights,  zones...).

However, using **packages** is recommended.

<div style="font-size: 75%;">
```yaml
  packages:
    devices: !include packages/devices.yaml
    media: !include packages/media.yaml
    tracking: !include packages/tracking.yaml
    ...
```
</div>

## Media Players, TVs...

Many players and TVs are supported, implementing several methods (services).

<div style="font-size: 75%;">
```yaml
media_player:
  - platform: clementine
    host: !secret laptop_ip

  - platform: lg_netcast
    #name: tvlg
    host: !secret tv_ip
    access_token: !secret tv_password
```
</div>

## Grouping and presentation

* Each group is presented as a panel in the UI.
* A group can be used as view using `view: True`.
* Default view can be overrided declaring a view with name `default_view`.
* Items not included in any group are shown in the default view.
* Use `group: !include groups.yaml` to split config.
* Groups are scriptable (`turn_on`, `turn_off`).

---

```yaml
media:
  name: Media

switches:
  name: Interruptores
  entities:
    - switch.pi_display

default_view:
  view: True
  name: Casa
  entities:
    - group.datetimeweather
    - group.media
    - group.switches
```

## History

Keeps historic data, and shows the *logbook* and *history* entries in the
frontend.

```yaml
# Recorder
recorder:
# Enables support for tracking state changes over time.
history:
# View all events in a logbook
logbook:
```
## Tracking

Device tracking can be achieved in several ways:

* Ping
* Network scanning
* Router integration
* Owntracks or GPSLogger

# Automation

## Components

![](./media/diagram-hub.png){style=float:right;width:50%;}

Components:

* Have state
* Trigger events
* Support actions

\
\
<div style="clear:both; font-size: 70%;">
_Component state can easily be inspected from the web interface._
</div>

## Architecture

![](./media/diagram-arch.png){style=float:right;width:38%;}

<div style="width: 60%;">
* **Event Bus**
* **State Machine**: Fires state change events.
* **Service Registry**: Registry of actions on components (`light.turn_on`).
* **Timer**: Fires time change events.
</div>

## Automation

Ingredients of a HASS automation:

1. Events that trigger the automation
2. Conditions (optional)
3. Commands / actions to run

---

![](./media/diagram-automation.png){style=float:right;width:100%;}

## Automation

```yaml
automation:
- alias: Turn on music on light
  trigger:
    platform: state
    entity_id: switch.kitchen_light
    from: 'off'
    to: 'on'
  action:
    - service: script.play_music
```

## Scripts

```yaml
script:

  play_music:
    sequence:

      #- condition: state
      #  entity_id: input_boolean.welcome_music
      #  state: 'on'

      - service: media_player.volume_set
        entity_id:
          - media_player.clementine_remote
        data_template:
          volume_level: "1.0"
      - service: media_player.media_play
        entity_id:
          - media_player.clementine_remote

      - delay: 00:00:08

      - service: tts.google_say
        entity_id: media_player.mpdlaptop
        data_template:
          message: "¡Hola Master Supremo! ¡Bienvenido!"
```

# Deployment

## Deployment Options (on rPi)

Some options already include OpenZWave, \
MQTT Broker ([Mosquitto](https://mosquitto.org/)), HDMI-CEC for Raspberry Pi.

- Raspbian all-in-one ([fabric](http://www.fabfile.org/))
- Hassbian
- Docker
- Custom scripts

> Do not forget about SSL!

## Develop components

```python
import asyncio
from datetime import timedelta
import logging
import time

import voluptuous as vol

import homeassistant.helpers.config_validation as cv
from homeassistant.components.media_player import (
    SUPPORT_NEXT_TRACK, SUPPORT_PAUSE, SUPPORT_PREVIOUS_TRACK, PLATFORM_SCHEMA,
    SUPPORT_VOLUME_STEP, SUPPORT_SELECT_SOURCE, SUPPORT_PLAY, MEDIA_TYPE_MUSIC,
    SUPPORT_VOLUME_SET, MediaPlayerDevice)
from homeassistant.const import (
    CONF_HOST, CONF_NAME, CONF_PORT, CONF_ACCESS_TOKEN,
    STATE_OFF, STATE_PLAYING, STATE_PAUSED, STATE_UNKNOWN)

REQUIREMENTS = ['python-clementine-remote==1.0.1']

DEFAULT_NAME = 'Clementine Remote'
DEFAULT_PORT = 5500

SUPPORT_CLEMENTINE = SUPPORT_PAUSE | SUPPORT_VOLUME_STEP | \
                     SUPPORT_PREVIOUS_TRACK | SUPPORT_VOLUME_SET | \
                     SUPPORT_NEXT_TRACK | \
                     SUPPORT_SELECT_SOURCE | SUPPORT_PLAY

PLATFORM_SCHEMA = PLATFORM_SCHEMA.extend({
    vol.Required(CONF_HOST): cv.string,
    vol.Optional(CONF_ACCESS_TOKEN, default=None): cv.positive_int,
    vol.Optional(CONF_NAME, default=DEFAULT_NAME): cv.string,
    vol.Optional(CONF_PORT, default=DEFAULT_PORT): cv.port,
})


def setup_platform(hass, config, add_devices, discovery_info=None):
    """Set up the Clementine platform."""
    from clementineremote import ClementineRemote
    host = config.get(CONF_HOST)
    port = config.get(CONF_PORT)
    token = config.get(CONF_ACCESS_TOKEN)

    client = ClementineRemote(host, port, token, reconnect=True)

    add_devices([ClementineDevice(client, config[CONF_NAME])])


class ClementineDevice(MediaPlayerDevice):
    """Representation of Clementine Player."""

    def __init__(self, client, name):
        """Initialize the Clementine device."""
        self._client = client
        self._name = name
        self._muted = False
        self._volume = 0.0
        self._track_id = 0
        self._last_track_id = 0
        self._track_name = ''
        self._track_artist = ''
        self._track_album_name = ''
        self._state = STATE_UNKNOWN

    def update(self):
        """Retrieve the latest data from the Clementine Player."""
        try:
            client = self._client

            if client.state == 'Playing':
                self._state = STATE_PLAYING
            elif client.state == 'Paused':
                self._state = STATE_PAUSED
            elif client.state == 'Disconnected':
                self._state = STATE_OFF
            else:
                self._state = STATE_PAUSED

            if client.last_update and (time.time() - client.last_update > 40):
                self._state = STATE_OFF

            self._volume = float(client.volume) if client.volume else 0.0

            if client.current_track:
                self._track_id = client.current_track['track_id']
                self._track_name = client.current_track['title']
                self._track_artist = client.current_track['track_artist']
                self._track_album_name = client.current_track['track_album']

        except:
            self._state = STATE_OFF
            raise

    @property
    def name(self):
        """Return the name of the device."""
        return self._name

    @property
    def state(self):
        """Return the state of the device."""
        return self._state

    @property
    def volume_level(self):
        """Volume level of the media player (0..1)."""
        return self._volume / 100.0

    @property
    def source(self):
        """Return  current source name."""
        source_name = "Unknown"
        client = self._client
        if client.active_playlist_id in client.playlists:
            source_name = client.playlists[client.active_playlist_id]['name']
        return source_name

    @property
    def source_list(self):
        """List of available input sources."""
        source_names = [s["name"] for s in self._client.playlists.values()]
        return source_names

    def select_source(self, source):
        """Select input source."""
        client = self._client
        sources = [s for s in client.playlists.values() if s['name'] == source]
        if len(sources) == 1:
            client.change_song(sources[0]['id'], 0)

    @property
    def media_content_type(self):
        """Content type of current playing media."""
        return MEDIA_TYPE_MUSIC

    @property
    def media_title(self):
        """Title of current playing media."""
        return self._track_name

    @property
    def media_artist(self):
        """Artist of current playing media, music track only."""
        return self._track_artist

    @property
    def media_album_name(self):
        """Album name of current playing media, music track only."""
        return self._track_album_name

    @property
    def supported_features(self):
        """Flag media player features that are supported."""
        return SUPPORT_CLEMENTINE

    @property
    def media_image_hash(self):
        """Hash value for media image."""
        if self._client.current_track:
            return self._client.current_track['track_id']

        return None

    @asyncio.coroutine
    def async_get_media_image(self):
        """Fetch media image of current playing image."""
        if self._client.current_track:
            image = bytes(self._client.current_track['art'])
            return (image, 'image/png')

        return None, None

    def volume_up(self):
        """Volume up the media player."""
        newvolume = min(self._client.volume + 4, 100)
        self._client.set_volume(newvolume)

    def volume_down(self):
        """Volume down media player."""
        newvolume = max(self._client.volume - 4, 0)
        self._client.set_volume(newvolume)

    def mute_volume(self, mute):
        """Send mute command."""
        self._client.set_volume(0)

    def set_volume_level(self, volume):
        """Set volume level."""
        self._client.set_volume(int(100 * volume))

    def media_play_pause(self):
        """Simulate play pause media player."""
        if self._state == STATE_PLAYING:
            self.media_pause()
        else:
            self.media_play()

    def media_play(self):
        """Send play command."""
        self._state = STATE_PLAYING
        self._client.play()

    def media_pause(self):
        """Send media pause command to media player."""
        self._state = STATE_PAUSED
        self._client.pause()

    def media_next_track(self):
        """Send next track command."""
        self._client.next()

    def media_previous_track(self):
        """Send the previous track command."""
        self._client.previous()

```

## A more complete example...

![](./media/hass-screenshot-01.png){width=70%}


# Final words...

## Pros and gotchas

Pros:

* Clean and useful frontend.
* Support for hundreds of devices.
* Free and Open Source.
* Runs on Raspberry Pi.
* Your data is not sent to third parties.

---

Gotchas:

* 3 configuration styles (per item, per platform, per package).
* Different typing of variables and states.
* Components inconsistent behavior, may cause race conditions.
* API and config is still changing.

## Future

* Not limited to *home*: greenhouses, workshops, customer premises, IoT.
* Privacy, security and availability considerations.
* Risk of loss of control (cloud services).

<aside class="notes">
    With the improvements in AI, a lot of tasks are now
    achievable. In the near future Google or Alexa will be able to pick
    up spoken instructions for automations. They might as well close
    their protocols and hamper interoperability.
</aside>

## References

* *Home Assistant*: [https://home-assistant.io/]()
* *This talk*: [https://github.com/jjmontesl/talk-hass-pydaygalicia2017]()
* *My config*: [https://github.com/jjmontesl/home-assistant-config]()

* [Development intro talk by Paulus Schoutsen](https://www.youtube.com/watch?v=Cfasc9EgbMU)

## Q&A ?

Thanks!

