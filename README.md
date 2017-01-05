# tellcore-remote-spy
Assist Tellstick users to configure remotes in /etc/tellstick.conf

Uses python library tellcore-py, which can be installed with
~~~~
pip3 install tellcore-py
~~~~
unless you already have it.

If you want your Tellstick Duo (and the telldusd daemon) to pick up your remotes (or magnetic door sensors) you need to add them to /etc/tellstick.conf with
an entry like this:
~~~~
device {
  id = 101
  name = "Sensor Front Door"
  protocol = "arctech"
  model = "codeswitch"
  parameters {
    house = "56981162"
    unit = "16"
  }
}
~~~~

The parameters (house and unit) can be found with the tellcore_events script (from tellcore-py), which can listen to raw events
picked up by the Tellstick Duo. 

~~~~
hacker@pannrummet:~$ tellcore_events --raw
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;method:turnon;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0010011010;method:turnoff;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;method:turnon;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0010011010;method:turnoff;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;method:turnon;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0010011010;method:turnoff;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:39041965;unit:14;group:0;method:turnoff;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0100110100;method:turnon;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;method:turnon;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0010011010;method:turnoff;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:5487533;unit:14;group:0;method:turnoff;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0100110100;method:turnon;
[RAW] 1 <- class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;method:turnon;
[RAW] 1 <- class:command;protocol:sartano;model:codeswitch;code:0010011010;method:turnoff;
~~~~

The above is the output after pressing the remote a couple of times. As you can see, each press on the remote sends several codes,
but not all codes are sent every time. If you want the remote to work properly, you should configure it with one of the codes that are always received. This script helps you identify which combination you should use. It captures all raw events and counts them so it's easier to see the codes that always work (both for "on" and "off" presses). After each event, it prints a list of the number of times each event has been received. Start it with <code>python3 tellcore-remote-spy</code> and press "on" and "off" on your remote a number of times. After a few presses, the output should look something like this:

~~~~
----
class:command;protocol:sartano;model:codeswitch;code:0100110100;: 1
class:command;protocol:arctech;model:selflearning;house:2743766;unit:3;group:1;: 1
class:command;protocol:arctech;model:selflearning;house:5487533;unit:14;group:0;: 1
class:command;protocol:arctech;model:selflearning;house:59406197;unit:10;group:1;: 1
class:command;protocol:sartano;model:codeswitch;code:1011101001;: 1
class:command;protocol:arctech;model:selflearning;house:2743766;unit:7;group:1;: 2
class:command;protocol:sartano;model:codeswitch;code:1001101001;: 2
class:command;protocol:sartano;model:codeswitch;code:0101110100;: 2
class:command;protocol:arctech;model:selflearning;house:10975021;unit:6;group:0;: 2
class:command;protocol:sartano;model:codeswitch;code:0010111010;: 9
class:command;protocol:sartano;model:codeswitch;code:0010011010;: 13
class:command;protocol:arctech;model:selflearning;house:10975066;unit:12;group:0;: 22
~~~~

From this output, it's clear that the correct entry in /etc/tellstick.conf should be the below, as it's captured 22 times in total:

~~~~
device {
  id = 200
  name = "My remote"
  protocol = "arctech"
  model = "selflearning-switch"
  parameters {
    house = "10975066"
    unit = "12"
  }
}
~~~~
Note that for the parameter "model", the value "selflearning" from the raw event should be written as "selflearning-switch" in tellstick.conf

After adding the above to /etc/tellstick.conf and restarting telldusd, your Duo will now receive events from your remote:
~~~~
hacker@pannrummet:~$ tellcore_events --device
[DEVICE] 200 -> turn on
[DEVICE] 200 -> turn off
~~~~
