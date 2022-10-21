# Inettest
 Bash script for test network connection

## Description

+ Consistently conduct two types of tests: ping and getting a page (or file) from a web server

+ Notify the user about errors or passed tests, with colored text.

+ Give sound signals about the successful completion of the test, or about an error on the PC-speaker.

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-main.png?raw=true)

For sound alerts, [PC-speaker is configured and the beep utility is installed](https://tolik-punkoff.com/2018/01/14/pc-speaker-v-linux-ili-kak-sdelat-beep-iz-konsoli-vstroennym-dinamikom-pk/) ([copy](https://lj.rossia.org/users/hex_laden/380240.html)).

The sequence of operations is specified in the configuration file.

## Configuration file

The configuration file consists of lines in the following format:

`operation|address|display description|abort/ignore`

So far, only two operations are supported:

`ping` - to ping an address

`getp` - to get a page or file from a WEB server (using wget)

address - address, WWW for `getp`, IP or WWW for `ping`.

`display description` - description of the operation, the text that will be displayed on the console.

`break/ignore` - if the operation fails, if the keyword `break` is specified, the script stops and displays a message that errors occurred during testing. If you specify a different value, such as `skip`, then the script will continue to run tests until the configuration file ends, or until the next test failure occurs, where the `break` option is specified.

Operation names and `break` are case insensitive (i.e. you can write `Ping`, `ping` or `Getp`, `GeTP` or `BreaK`, `breaK`).

The first two fields (operation and address) are required. If they are not specified, the script will break on the line with an error:

`...`

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[DATA EXPUNGED]|Provider IP|break`

`getp`

`...`

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-config-error.png?raw=true)

If no description field is specified, it will default to No desription:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-no-desription.png?raw=true)

If the last field is not filled, then it takes the value break.

If the first field contains an incorrectly specified operation code, the script will ignore it, displaying an appropriate message, continue to perform other operations, but will end with an error:

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[DATA EXPUNGED]|Provider IP|break`

`getp|[DATA EXPUNGED]|Provider page`

`1234|[DATA EXPUNGED]|VPN Server IP|break`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-unknow-operation.png?raw=true)

The `|` character is used to separate fields

You can use comments in the configuration file that begin with the `#` character, everything after this character is ignored. Blank lines are also ignored.

## Sample config file (without data)

`#inettest.cfg example`

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|x.x.x.x|Provider IP|break #Change x.x.x.x to your provider ip`

`getp|myprovider.net|Provider site page|skip #change myprovider.net to real site your provider`

`ping|x.x.x.x|VPN Server IP|break #Change x.x.x.x to real VPN provider IP`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

## Script variables

The script has no command line parameters, the main settings are made through the variables of the script itself:

`CONFIG` - path to the configuration file, for example `CONFIG="./inettest.cfg"`. If the configuration file is not found, the script will throw an error:

`CRITICAL ERROR: Config file ./inettest.cfg not exist!`

`NOCOLOR` - if the value is `0`, enable output of colored text to the console, if `1` - disable. Default `0`

`NOCOLOR=1`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-no-color.png?raw=true)

`NOSOUNDP` - enable (default `0`) or disable (`1`) sound during tests. A sound is emitted after each individual test.

`NOSOUNDF` - similar to the previous variable, only the sound sounds after the end of all tests or their interruption.

`NOADDR` - `0`, include the tested address in the script output, `1` - do not include.

`NOADDR=0`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-noaddr-0.png?raw=true)

`NOADDR=1`:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-main.png?raw=true)

`PACKETS` - number of packets for `ping` command (default `PACKETS=3`)

`TIMEOUT` - timeout for getting a page or a file (in the script it is done using `wget`, by default `TIMEOUT=5`)

## Test on network error

Config:

`Ping|192.168.0.1|Main router|break #Main router ping`

`ping|[DATA EXPUNGED]|Provider IP|skip`

`getp|[DATA EXPUNGED]Provider page|skip`

`ping|[DATA EXPUNGED]|VPN Server IP|break`

`ping|8.8.8.8|Internet IP|break`

`getp|google.com|Internet page|break`

Result:

![screen](https://github.com/tolik-punkoff/Inettest/blob/main/screens/inettest-network-errors.png?raw=true)

## Error codes

`ping`:

1 - No reply (not one of the packets reached the pinged address)

2 - Other error (another error, in most cases - "network unavailable").

`getp` (`wget`):

1 - Other / general error (generic error code)

2 - Error in command line options or configuration files (.wgetrc or .netrc)

3 - File input / output error (I / O error)

4 - Network error (for example, when the connection is broken)

5 - SSL Error

6 - Authentication error (wrong username or password)

7 - Protocol error

8 - Server error (for example, the required file was not found on the server, error 404)

## Script return codes

0 - No errors occurred during the tests.

1 - At least one error has occurred.