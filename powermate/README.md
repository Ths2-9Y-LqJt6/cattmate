powermate
=========

This is a copy of Stef Nelson-Lindall's (aka bethebunny's) 
[powermate repo](https://github.com/bethebunny/powermate). 


A small python framework for scripting interactions with the Griffin Powermate.
- Python 2 and 3
- Any number of powermates
- Can have more than one script interact with the same powermate

setup
=====

In order to read and write to the Powermate event files on linux, you will need
to do the following (ymmv, but this should work on most modern distros).

```shellsession
$ sudo groupadd input
$ sudo usermod -a -G input "$USER"
$ echo 'KERNEL=="event*", NAME="input/%k", MODE="660", GROUP="input"' | sudo tee -a /etc/udev/rules.d/99-input.rules
```

After a reboot your scripts should be able to read/write to the device.

writing your own
================

```python
from powermate import PowerMateBase, LedEvent, MAX_BRIGHTNESS

class ExamplePowerMate(PowerMateBase):
  def __init__(self, path):
    super(ExamplePowerMate, self).__init__(path)
    self._pulsing = False
    self._brightness = MAX_BRIGHTNESS

  def short_press(self):
    print('Short press!')
    self._pulsing = not self._pulsing
    print(self._pulsing)
    if self._pulsing:
      return LedEvent.pulse()
    else:
      return LedEvent(brightness=self._brightness)

  def long_press(self):
    print('Long press!')

  def rotate(self, rotation):
    print('Rotate {}!'.format(rotation))
    self._brightness = max(0, min(MAX_BRIGHTNESS, self._brightness + rotation))
    self._pulsing = False
    return LedEvent(brightness=self._brightness)

  def push_rotate(self, rotation):
    print('Push rotate {}!'.format(rotation))

if __name__ == '__main__':
  pm = ExamplePowerMate(glob.glob('/dev/input/by-id/*PowerMate*')[0])
  pm.run()
```

You can implement any subset of these commands. By default a press will take
1 second to be considered a long\_press, and a long\_press will default to
doing a short\_press unless overridden. The threshold for a long press may
be changed by passing the keyword argument long\_threshold to the super
constructor.

If you have several powermates that you want to control you will need to modify
the globbing expression in the main block appropriately.

If you want to have more than one script or rules sets that interact with a
single powermate, they will all need to be run through one central
PowerMateBase instance; otherwise events read by one script won't be read by
the other. This is convenient to do through the add\_listener method.
See the example at the bottom of powermate.py for how to do this. This should
be robust to badly behaving code, so eg. if one of your handlers is hanging it
won't stop the other handlers from working properly.
