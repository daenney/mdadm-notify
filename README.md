# Desktop notifications for mdadm

The `mdadm-notify` script is a little program that handles displaying
notifications from `mdadm --monitor` by using the desktop notification system.
To do so you have to have the `notify-send` binary available in your path,
which usually comes with `libnotify` as well as a few other basic utilities
that are documented in the script (they should be readily available on most
Linux distributions as part of the base installation).

It handles the following notifications (from `mdadm(8)`):

| Notification | Urgency |
|--------------|----------|
|`DeviceDisappeared`| critical |
|`RebuildStarted`| normal |
|`RebuildNN`| normal |
|`RebuildFinished` | normal |
|`Fail`| critical |
|`FailSpare`| critical |
|`SpareActive`| normal |
|`NewArray`| normal
|`DegradedArray` | critical |
| `MoveSpare` | normal |
| `SparesMissing` | critical |
| `TestMessage` | low |

All messages that are critical will never expire and thus require explicit
interaction to dismiss them. All other messages have a timeout of 3 seconds,
so the overlay will disappear but you can still find them in the notification
area in case you missed them.

## Installation

* Put the script somewhere, ensure it's owned by root and executable
* Set the `PROGRAM` option in `mdadm.conf` to point to this script
* (Re)start the mdadm or mdmonitor service

## Usage

Sit back and relax, you'll see notifications pop up like:
![notification-rebuild](screenshots/notification-rebuild.png "Rebuild notification")
