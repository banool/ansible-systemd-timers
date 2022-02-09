# ansible-systemd-timer

## About
This roles enables you to create systemd timers which call scripts or execute commands.

## Usage

Define a variable ```timers```. This variable is a dictionary. Every key is a new timer.

### Example

Here is an example for my 1337 Telegram Bot. The Timer "calls" a script which sends the message "It's now 13:37" in one of my Telegram chats every day at 13:37 GMT o'Clock.

```
timers:
   1337TelegramBot:
      user: telegrambot
      ExecStart: /home/telegrambot/sendMessage.pl
      OnCalendar: "*-*-* 13:37:00 CET"
      AccuracySec: 5s
```

That's all the magic.

### Existing variables

| Variable | Required |  Default value / Explanation |
|----------|----------|------------------------------|
| userspace | yes | Takes a boolean argument. If true, install the unit / timer to `/home/$USER/.config/systemd/user/`, otherwise install to `/etc/systemd/system/`. |
| ExecStart | yes | Which command or script to execute |
| User | no | Under which users the command is executed. Default: root (for `userspace=false`) or `$USER` for (`userspace=true`) |
| Persistent | no | Takes a boolean argument. If true, the time when the service unit was last triggered is stored on disk. When the timer is activated, the service unit is triggered immediately if it would have been triggered at least once during the time when the timer was inactive. This is useful to catch up on missed runs of the service when the machine was off. Note that this setting only has an effect on timers configured with OnCalendar=. Defaults to false. [Source](https://www.freedesktop.org/software/systemd/man/systemd.timer.html) |
| OnActiveSec | no | Relative time after the timer unit was last activated |
| OnBootSec | no | Relative time after the computer was booted |
| OnStartupSec | no | Relative time after systemd was started |
| OnUnitActiveSec | no | Relative time after the service unit was last activated |
| OnUnitInactiveSec | no | Relative time after the service unit was last deactivated |
| OnCalendar | no | Absolute time when to call activate the unit |
| AccuracySec | no | Timer have a default accuracy of round about one minute. You can set the accuracy with this var. Default: 15s |
| Restart | no | Configures whether the service shall be restarted when the service process exits, is killed, or a timeout is reached |
| RestartSec | no | Configures the time to sleep before restarting a service |
| ExecStartPre | no | Additional commands that are executed before the command in ExecStart= |
| ExecStartPost | no | Additional commands that are executed after the command in ExecStart= |

You can chain every On* variable. Example:

```
timers:
   updateDNS:
      userspace: false
      ExecStart: /home/dnsupdate/updateMe.pl
      User: dnsupdate
      OnStartupSec: 20s
      OnUnitActiveSec: 5m
```

The timer unit will be triggered 20 seconds after systemd was started and then every 5 minutes.

More about timers: https://www.freedesktop.org/software/systemd/man/systemd.timer.html

More about timespans: https://www.freedesktop.org/software/systemd/man/systemd.time.html

## Working with shell redirection

Shell redirection does not work out of the box. You have to work around that by calling `sh` or `bash`.   
This won't work: `echo hello > /var/log/hello.log`
This will work: `/usr/bin/bash -c \"echo hello > /var/log/hello.log\"`

Tip: Always use full paths. To see where `sh` or `bash` is stored on your system you have to use `which`:

```
[root@pizza ~]# which bash
/usr/bin/bash
```
