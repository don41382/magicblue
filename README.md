# magicblue - Python script and library to control Magic Blue bulbs over bluetooth


The Magic Bulb is, as far as I know, the cheapest bluetooth RGB light bulb on the market : you can get it for as low as ~8€/9$ on sites like
[Gearbest](http://www.gearbest.com/smart-light-bulb/pp_230349.html). It works pretty good and comes with mobile apps.

Unfortunately I haven't found any API or documentation for it, which is why I started this project.

I haven't fully retro-engineered the protocol yet so it's not complete but
[Characteristics list page](https://github.com/Betree/pyMagicBlue/wiki/Characteristics-list) and
[How to use manually with Gatttool page](https://github.com/Betree/pyMagicBlue/wiki/How-to-use-manually-with-Gatttool)
should give you enough details to start working on your own implementation if you need to port this for another
language / platform.
On the [research/bluetooth branch](https://github.com/Betree/pyMagicBlue/tree/research/bluetooth) you'll also find capture of bluetooth packets exchanged
between Android and the bulb (open hci_capture.log with Wireshark).

Tested on Linux only. I'll be happy to get your feedback on other platforms !

## Installation
### Linux
You must use python 3+ and have a proper Bluetooth 4.0 interface installed on your machine.

    sudo apt-get install libbluetooth-dev
    pip install git+https://github.com/Betree/pyMagicBlue.git

### Raspberry Pi
Gattlib may cause you some troubles if you're trying to set this project up for a Raspberry Pi. Fortunately [Mark Otting](https://github.com/b0tting) proposed a solution for this :

    Pip started installing gattlib, then handed something over to gcc and that crapped out,
    often "freezing" the Pi. It turns out that whatever it was that gattlib needed compiling
    ran the pi out of memory. The solution was to increase swap space to 512mb (or 1024mb on
    a 256mb memory RPI. Also note that Raspbian uses dphys-swapfile). It takes 30 minutes or
    so but succesfully compiled after that.

## Usage

**Library needs root permissions to use Bluetooth features.**

If you run into problems during devices listing or connect, try to follow this procedure to ensure your Bluetooth interface works correctly : [How to use manually with Gatttool page](https://github.com/Betree/pyMagicBlue/wiki/How-to-use-manually-with-Gatttool)

### Using it as an API

    >>> from magicbluelib import magicblue
    
    >>> bulb_mac_address = 'XX:XX:XX:XX:XX:XX'
    >>> bulb = MagicBlue(bulb_mac_address)
    >>> bulb.connect()
    >>> bulb.set_color([255, 0, 0])         # Set red
    >>> bulb.set_random_color()             # Set random
    >>> bulb.turn_off()                     # Turn off the light
    >>> bulb.turn_on()                      # Set white light

### Using it as a tool
Script must be run as root.

You can always specify which bluetooth adapter (default: hci0) you want to use by specifying it with the -a option. 

#### Using the interactive shell
Just launch magicblueshell as root user :

    piouffb@Ordinatron-3000:~/workspace/python/pyMagicBlue$ sudo magicblueshell 
    Magic Blue interactive shell v0.1
    Type "help" to see what you can do
    > help
     ----------------------------
    | List of available commands |
     ----------------------------
    COMMAND         PARAMETERS               DETAILS
    -------         ----------               -------
    help                                     Show this help
    list_devices                             List Bluetooth LE devices in range
    connect         mac_address              Connect to light bulb
    disconnect                               Disconnect from current light bulb
    set_color       name|hexvalue            Change bulb's color
    set_warm_light  intensity[0.0-1.0]       Set warm light
    turn            on|off                   Turn on / off the bulb
    exit                                     Exit the script

    > list_devices
    Listing Bluetooth LE devices in range. Press CTRL+C to stop searching.
    Name                Mac address 
    ----                ----------- 
    LEDBLE-1D433903     C7:17:1D:43:39:03
    ---------------
    ^C
    
    > connect C7:17:1D:43:39:03
    INFO:__main__:Connected : True
    > exit
    Bye !

#### Passing command as an option
Script can also be used by command line (for example to include it in custom shell scripts)
Usage is defined as follow :

    usage: magicblueshell [-h] [-l LIST_COMMANDS] [-c COMMAND] [-m MAC_ADDRESS]
    
    optional arguments:
      -h, --help            show this help message and exit
      -l LIST_COMMANDS, --list_commands LIST_COMMANDS
                            List available commands
      -c COMMAND, --command COMMAND
                            Command to execute
      -m MAC_ADDRESS, --mac_address MAC_ADDRESS
                            Device mac address. Must be set if command given in -c needs you to be connected
                            
So if you want to change the color of bulb with mac address "C7:17:1D:43:39:03", just run :
    
    sudo magicblueshell -c 'set_color red' -m C7:17:1D:43:39:03
