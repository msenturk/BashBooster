Bash Booster

Bash Booster is a single file library, which provides various features useful during setup environment and preparing servers. It is inspired by [Chef](http://www.chef.io/) and was developed to be used with [Vagrant](http://vagrantup.com/). When Chef is too heavy, use Bash Booster, because it has been written using Bash only and **requires nothing**.

It also shipped with task runner utility, so you can install into your system and [use as an automation tool](#task-runner).

Table of Contents

*   [Quick Start](#quick-start)
*   [Philosophy](#philosophy)
*   [Code Organization](#code-organization)
*   [Module Description](#module-description)
    *   [error](#error)
    *   [var](#var)
    *   [log](#log)
    *   [exit](#exit)
    *   [assert](#assert)
    *   [ext](#ext)
    *   [exe](#exe)
    *   [workspace](#workspace)
    *   [tmp](#tmp)
    *   [template](#template)
    *   [properties](#properties)
    *   [event](#event)
    *   [download](#download)
    *   [flag](#flag)
    *   [read](#read)
    *   [sync](#sync)
    *   [wait](#wait)
    *   [iptables](#iptables)
    *   [task](#task)
    *   [apt](#apt)
    *   [yum](#yum)
    *   [brew](#brew)
    *   [augeas](#augeas)
*   [Task Runner](#task-runner)
*   [Support & Feedback](#support-feedback)
*   [Contribution](#contribution)
*   [License](#license)
*   [Changes](#changes)
    *   [0.6 (2019-02-21)](#06-2019-02-21)
    *   [0.5beta (2017-12-14)](#05beta-2017-12-14)
    *   [0.4beta (2016-09-20)](#04beta-2016-09-20)
    *   [0.3beta (2015-03-07)](#03beta-2015-03-07)
    *   [0.2beta (2014-10-11)](#02beta-2014-10-11)
    *   [0.1beta5 (2014-10-10)](#01beta5-2014-10-10)
    *   [0.1beta4 (2014-09-08)](#01beta4-2014-09-08)
    *   [0.1beta3 (2014-08-20)](#01beta3-2014-08-20)
    *   [0.1beta2 (2014-07-23)](#01beta2-2014-07-23)
    *   [0.1beta (2014-07-16)](#01beta-2014-07-16)

## Quick Start[¶](#quick-start "Permanent link")

Download [ready to use library archive](https://bitbucket.org/kr41/bash-booster/downloads) or...

1.  Get the source code:

    $ hg clone https://bitbucket.org/kr41/bash-booster bash-booster
    $ cd bash-booster

2.  Build the library:

    $ ./build.sh

3.  Get the library at `build/bashbooster.sh` and use it!

Note for OS X users. OS X is shipped with an old version of Bash, so you need to get a new one using [Homebrew](http://brew.sh/).

$ brew install bash

A traditional “Hello World” script looks like this (you can find it at `examples/helloworld.sh`):

#!/usr/bin/env bash

\# Remove undesirable side effects of CDPATH variable
unset CDPATH
\# Change current working directory to the directory contains this script
cd "$( dirname "${BASH\_SOURCE\[0\]}" )"

\# Initialize Bash Booster
source build/bashbooster.sh

\# Log message with log level "INFO"
bb-log-info "Hello World"

It just prints a line to `stderr`:

helloworld.sh \[INFO\] Hello world

More interesting example, which demonstrates almost all features of Bash Booster you can find at `examples/vagrant/bootstrap.sh`. This script is used for provisioning virtual machines managed by [Vagrant](http://vagrantup.com/). A `Vagrantfile` placed at `examples/vagrant` sets up three virtual machines: ubuntu, centos, and debian. Bootstrap script installs Nginx web-server, builds Bash Booster documentation, and places compiled HTML into web-root directory. Just run:

$ cd examples/vagrant
$ vagrant up

...and have some coffee, then visit:

*   [http://localhost:8081](http://localhost:8081)—Ubuntu machine
*   [http://localhost:8082](http://localhost:8082)—CentOS machine
*   [http://localhost:8083](http://localhost:8083)—Debian machine

If you run `vagrant provision` again, script will finish almost immediately. It happens, because it does not do unnecessary job: all packages installed, web-server configured, HTML compiled.

## Philosophy[¶](#philosophy "Permanent link")

The main goal of Bash Booster is ability to write [idempotent](http://en.wikipedia.org/wiki/Idempotence) scripts. For instance, you have to manage developer’s virtual machine using Ubuntu. At the start of your project you just need a web-server installed and nothing more. But requirements may be changed in future. So you place `bootstrap.sh` script at the root of your project sources:

#!/usr/bin/env bash

apt-get update
apt-get install nginx

Each time you pull the code from repository, you have to run this script on the virtual machine, because someone from you team might update the requirements and add some other packages to install. I think, you will automate this, so the script will run at VM start up time. And at most of the time it will just make you to wait for `apt-get update` command. It is annoying.

Once you will think about replacing the tool. You may think about Chef. To run it you will have to install Ruby. But Ruby at the Ubuntu repositories has an ancient version, which does not support Chef. So you need to install RVM and...

Wait, what the heck? You was just going to install Nginx, why do you need all this stuff? The answer is: you don’t. You need a set of handy Bash functions, which requires nothing, but only Bash, which already included into each Linux distribution. So Bash Booster is such set. The script above can look like this:

#!/usr/bin/env bash

unset CDPATH
cd "$( dirname "${BASH\_SOURCE\[0\]}" )"

source bashbooster.sh

\# The command bellow will check whether "nginx" package already installed.
\# If it doesn't, it will install it.
\# And it will also update Apt cache, before installation.
\# If Nginx already installed, it will do nothing.
bb-apt-install nginx

## Code Organization[¶](#code-organization "Permanent link")

Bash Booster comes with a set of modules. These modules are merged to a single file (by `build.sh` script) to be easy to use. But their sources are placed at `source` directory to be easy to read, because source code is the best documentation. Each module has a numeric index, which indicates inclusion order. For instance, workspace management module [`10_workspace.sh`](#workspace) will be included before events management one [`20_event.sh`](#event).

Each function has the following name format: `bb-module-func`, where `bb` is a common function prefix (means “Bash Booster”), `module` is module name, `func` is function name. For example, function [`bb-event-on`](#bb-event-on) (subscribes handler on event) from [`event`](#event) module. Some functions does not contain `func` part of the name. For example, [`bb-exit`](#bb-exit) function (terminates script with specified exit code and logs exit message) from module [`exit`](#exit). Boolean functions end in Ruby style by question mark. For example, [`bb-yum?`](#bb-yum) function returns `0` if Yum (the default package manager used in CentOS) is available and `1` otherwise.

There is special function names: `init` and `cleanup`. They are used for module initialization and cleaning up its resources. You do not need to use these functions in your scripts. They are called automatically. So their description are not included into this document.

Each variable has a format, which is the similar to the function names: `BB_MODULE_VAR`.

There is also a special module `init`, which initialize Bash Booster and sets up a trap on `EXIT` signal. So **do not use** in your script:

trap my-cleanup-command EXIT

It will break cleanup process. [Subscribe](#bb-event-on) on `bb-cleanup` event instead, it will be fired just before exit:

bb-event-on bb-cleanup my-cleanup-command

## Module Description[¶](#module-description "Permanent link")

*   [error](#error)
*   [var](#var)
*   [log](#log)
*   [exit](#exit)
*   [assert](#assert)
*   [ext](#ext)
*   [exe](#exe)
*   [workspace](#workspace)
*   [template](#template)
*   [properties](#properties)
*   [event](#event)
*   [download](#download)
*   [flag](#flag)
*   [read](#read)
*   [sync](#sync)
*   [wait](#wait)
*   [task](#task)
*   [apt](#apt)
*   [yum](#yum)

### error[¶](#error "Permanent link")

The module contains a single function for handling errors

**bb-error?**

The function will return `true`, if previous operation fails, i.e. returns non-zero exit code. It also saves that exit code into global variable `BB_ERROR`. Example:

false
if bb-error?
then
    bb-log-error "An error with code $BB\_ERROR occured"
    return $BB\_ERROR
fi

The example above is equal to:

false
BB\_ERROR\=$?
if (( $BB\_ERROR !\= 0 ))
then
    bb-log-error "An error with code $BB\_ERROR occured"
    return $BB\_ERROR
fi

### var[¶](#var "Permanent link")

The module contains a single function for management undefined variables

**bb-var** VAR\_NAME DEFAULT\_VALUE

The function will set up variable `VAR_NAME` to `DEFAULT_VALUE`, if variable is undefined. It is used for configurable variables. For example:

#!/usr/bin/env bash

unset CDPATH
cd "$( dirname "${BASH\_SOURCE\[0\]}" )"

\# Change default location of workspace directory
BB\_WORKSPACE\="/var/myworkspace"
source bashbooster.sh

You can use this function to configure your own scripts using environment variables. For instance:

$ export MY\_VAR="Special Value"
$ ./my\_script.sh

Script `my_script.sh`:

#!/usr/bin/env bash

unset CDPATH
cd "$( dirname "${BASH\_SOURCE\[0\]}" )"

source bashbooster.sh
bb-var MY\_VAR "Default Value"

\# Do something useful

### log[¶](#log "Permanent link")

The module provides functions to log messages to `stderr`.

**BB\_LOG\_LEVEL**

Log verbosity level, default is `INFO`. This variable can be set to numeric or string values, i.e. 1–4, `DEBUG`, `INFO`, `WARNING`, or `ERROR`. Current log level can be gotten using functions [`bb-log-level-code`](#bb-log-level-code) and [`bb-log-level-name`](#bb-log-level-name).

**BB\_LOG\_PREFIX**

Log prefix, default is `"$( basename "$0" )"`, i.e. script name.

**BB\_LOG\_TIME**

Command to get date and time of log message, default is `date +"%Y-%m-%d %H:%M:%S"`.

**BB\_LOG\_FORMAT**

Log string format, default is `'${PREFIX} [${LEVEL}] ${MESSAGE}'`. The following variables can be used in log format:

*   `LEVEL_CODE`—Log level numeric value
*   `LEVEL`—Log level text value
*   `MESSAGE`—Message to log
*   `PREFIX`—Log message prefix, usually is [`BB_LOG_PREFIX`](#BB_LOG_PREFIX). If logger is called within Bash Booster function, prefix will be equal to its module name.
*   `TIME`—Log time, the output of [`BB_LOG_TIME`](#BB_LOG_TIME) command.
*   `COLOR`—Escape code to start colored output
*   `NOCOLOR`—Escape code to stop colored output

**BB\_LOG\_USE\_COLOR**

Boolean value, default is `false`. If set to `true` before Bash Booster initialization, [`BB_LOG_FORMAT`](#BB_LOG_FORMAT) will be wrapped by `COLOR` and `NOCOLOR` values, so that log output will be colored according to log level:

*   `DEBUG`—gray
*   `INFO`—green
*   `WARNING`—orange
*   `ERROR`—red

Changing this variable after initialization will take no effect.

**bb-log-level-code**

Prints to `stdout` current log level code, i.e. 1–4.

**bb-log-level-name**

Prints to `stdout` current log level name, i.e. `DEBUG`, `INFO`, `WARNING`, or `ERROR`.

**bb-log-debug** MESSAGE

Logs `MESSAGE` with `DEBUG` level.

**bb-log-info** MESSAGE

Logs `MESSAGE` with `INFO` level.

**bb-log-warning** MESSAGE

Logs `MESSAGE` with `WARNING` level.

**bb-log-error** MESSAGE

Logs `MESSAGE` with `ERROR` level.

**bb-log-deprecated** ALTERNATIVE \[CURRENT\]

Logs deprecation warning message: `"'$CURRENT' is deprecated, use '$ALTERNATIVE' instead"`. If optional `CURRENT` function name is not passed, it will be detected using callstack.

The function is mostly useful for Bash Booster developers.

### exit[¶](#exit "Permanent link")

**bb-exit** CODE MSG

Terminates script with status `CODE` and logs the message `MSG`. If `CODE` is equal to `0`, message will be logged with `INFO` level. If `CODE` is non-zero, message will be logged with `ERROR` level. Additionally, it will log call stack with `DEBUG` level. Usage:

bb-exit 1 "Something went wrong"

or:

bb-exit 0 "Success"

**bb-exit-on-error** MSG

If previous operation fails (returns non-zero exit code), the function will terminate script with the same code and given error message `MSG`. Usage:

false
bb-exit-on-error "Something went wrong"

It is equal to combination of [`bb-error?`](#bb-error) and [`bb-exit`](#bb-exit) functions:

false
if bb-error?
then
    bb-exit $BB\_ERROR "Something went wrong"
fi

### assert[¶](#assert "Permanent link")

**bb-assert** ASSERTION \[MSG\]

Evaluates `ASSERTION`. If assertion returns non-zero code, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"Assertion error '$ASSERTION'"`.

**bb-assert-root** \[MSG\]

Evaluates if the script is running as root. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"This script must be run as root!"`.

**bb-assert-file** FILE \[MSG\]

Evaluates if the file `FILE` exists. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"File '$FILE' not found"`.

**bb-assert-file-readable** FILE \[MSG\]

Evaluates if the file `FILE` is readable. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"File '$FILE' is not readable"`.

**bb-assert-file-writeable** FILE \[MSG\]

Evaluates if the file `FILE` is writeable. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"File '$FILE' is not writeable"`.

**bb-assert-file-executable** FILE \[MSG\]

Evaluates if the file `FILE` is executable. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"File '$FILE' is not executable"`.

**bb-assert-dir** DIR \[MSG\]

Evaluates if the directory `DIR` exists. If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"Directory '$DIR' not found"`.

**bb-assert-var** VAR \[MSG\]

Evaluates if the variable `VAR` is set (not empty). If assertion is false, it will exit script with code `3` and error message `MSG`. If `MSG` is not passed, it will use default one: `"Variable '$VAR' not set"`.

### ext[¶](#ext "Permanent link")

Some tasks can be easily solved using other scripting languages. This module provides features to add extension functions using short non-bash scripts. At the moment, only [Python](https://www.python.org) is available. However, it is good place for [adding](#contribution) other interpreters.

**bb-ext-python** NAME <BODY

Creates new function `NAME` using [Python](https://www.python.org) interpreter. Example:

bb-ext-python 'hello' <<EOF
import sys
print('Hello %s' % sys.argv\[1\])
EOF

hello 'World'   \# Prints: Hello World

**bb-ext-augeas** NAME <BODY

Creates new function `NAME` using [Augeas](http://augeas.net) interpreter. Example:

bb-ext-augeas 'set-ssh-port' <<EOF
set /files/etc/ssh/sshd\_config/Port 222
save
EOF

set-ssh-port    \# Sets "Port 222" in /etc/ssh/sshd\_config

The variable `BB_AUGEAS_PARAMS` can be used to provide additional parameters to the invocation of the [Augeas](http://augeas.net) interpreter.

The variable `BB_AUGEAS_ROOT` stores the directory to be used as the root by [Augeas](http://augeas.net). The default value is "/".

### exe[¶](#exe "Permanent link")

**bb-exe?** EXE

Checks whether executable `EXE` is available. It is a shortcut for `type -t "$EXE" > /dev/null`. Usage:

if ! bb-exe? pip
then
    GET\_PIP\="$( bb-download https://bootstrap.pypa.io/get-pip.py )"
    python "$GET\_PIP"
fi

### workspace[¶](#workspace "Permanent link")

The module manages workspace directory. It provides single variable for your scripts.

**BB\_WORKSPACE**

The variable stores full path to the workspace directory. The workspace directory is created on startup and deleted (if it is empty) on cleanup automatically.

The default value is `.bb-workspace`, which means the workspace will be created in the same directory, where caller script is stored. To override default value use:

#!/usr/bin/env bash

unset CDPATH
cd "$( dirname "${BASH\_SOURCE\[0\]}" )"

\# Change default location of workspace directory
BB\_WORKSPACE\="/var/myworkspace"
source bashbooster.sh

You can use relative path also. It will be unfolded to full one after initialization.

It is an appropriate place to store files, which are used by your script. Bash Booster itself uses this directory to store: [temp files](#tmp) at `$BB_WORKSPACE/tmp/`, [downloads](#download) at `$BB_WORKSPACE/download/`, and [flags](#flag) at `$BB_WORKSPACE/flag/`.

### tmp[¶](#tmp "Permanent link")

The module manages temporary files and directories. All files and directories created by the following functions will be automatically deleted on exiting script.

**bb-tmp-file**

Creates temporary file:

MY\_TMP\_FILE\="$( bb-tmp-file )"
echo "Some stuff" > "$MY\_TMP\_FILE"

**bb-tmp-dir**

Creates temporary directory:

MY\_TMP\_DIR\="$( bb-tmp-dir )"
touch "$MY\_TMP\_DIR/file1"
touch "$MY\_TMP\_DIR/file2"

### template[¶](#template "Permanent link")

Provides stupid and simple Bash-based templates handling. It is useful for variable substitution only, but in the most cases it is enough. If you are looking for something more powerful, you will have to install it by your own.

**bb-template** TEMPLATE\_FILE

Renders template from `TEMPLATE_FILE` to `stdout` using all defined variables.

Template file `$BB_WORSPACE/example.bbt`:

x=$(( A + B ))
msg='${MESSAGE}'

Script:

A\=1
B\=2
MESSAGE\='Hello World'
bb-template "$BB\_WORSPACE/example.bbt" > "$BB\_WORSPACE/example.txt"

Result output file `$BB_WORKSPACE/example.txt`:

x=3
msg='Hello World'

### properties[¶](#properties "Permanent link")

**NOTE,** the module is deprecated, use [`read`](#read) module instead.

**bb-properties-read** FILENAME \[PREFIX\]

See [`bb-read-properties`](#bb-read-properties).

### event[¶](#event "Permanent link")

The module provides functions to work with events. Typical use case is to make some job conditionally. For example, the following code pulls application sources from repository, rebuilds one, and reloads application server:

bb-event-on reload-server on-reload-server
on-reload-server() {
    bb-log-info "Reloading server"
    \# ...
}

bb-event-on rebuild-app on-rebuild-app
on-rebuild-app() {
    bb-log-info "Rebuilding application"
    \# ...
    bb-event-delay reload-server
}

cd "$PATH\_TO\_REPOSITORY"
git pull
bb-sync-dir "$BB\_WORKSPACE/sources" "$PATH\_TO\_REPOSITORY/sources" rebuild-app
bb-sync-file "/etc/server/config" "$PATH\_TO\_REPOSITORY/conf/server" reload-server

If source code of application is changed, it will rebuild application and reload server. If server configuration is changed, it will just reload server. Learn mode about functions [`bb-sync-dir`](#bb-sync-dir) and [`bb-sync-file`](#bb-sync-file) in [`sync`](#sync) module description.

There is also a special event `bb-cleanup`. This event fires automatically just before script termination.

**bb-event-on** EVENT HANDLER

Subscribes `HANDLER` on `EVENT`. `HANDLER` will be subscribed only once, so the second call with the same arguments will take no effect.

**bb-event-off** EVENT HANDLER

Removes `HANDLER` from `EVENT`.

**bb-event-fire** EVENT \[ARGUMENTS...\]

Fires `EVENT`. It will call all `EVENT` handlers with `ARGUMENTS` (if any) immediately. This function is not very useful in your scripts, it is mostly for internal usage.

**bb-event-delay** EVENT \[ARGUMENTS...\]

Delays `EVENT` to the end of script. It will call all `EVENT` handlers with `ARGUMENTS` during the cleanup process. Delayed event handlers can call this function too. If event is delayed twice with the same arguments, its handler will be called only once.

### download[¶](#download "Permanent link")

The module manages download directory and its contents.

**BB\_DOWNLOAD\_WGET\_OPTIONS**

As the variable name says, it stores additional [Wget](https://www.gnu.org/software/wget/) options and can be used to tune [`bb-download`](#bb-download) behavior.

**bb-download** URL \[TARGET \[FORCE\]\]

Downloads file from `URL` and writes it to `$BB_WORKSPACE/download/$TARGET`. The second argument `TARGET` can be omitted. In that case it will be detected using `basename "$URL"` command. If `TARGET` file already exists, the function will not download it again. Pass `true` as a `FORCE` argument to change this behavior. The full path to downloaded file will be printed into `stdout`. Usage:

MY\_FILE\="$( bb-download http://example.com/my\_file.txt )"
\# "$MY\_FILE" == "$BB\_WORKSPACE/download/my\_file.txt"

**bb-download-clean**

Removes all downloaded files, i.e. deletes directory `$BB_WORKSPACE/download`.

### flag[¶](#flag "Permanent link")

Some operations are not idempotent. And you need to save information, that some action has been done. This module provides functions for such use cases.

**bb-flag?** FLAG

Returns `0` if `FLAG` is set, and `1` otherwise. Usage:

if ! bb-flag? somestate
then
    \# Do something useful
    bb-flag-set somestate
fi

if bb-flag? someotherstate
then
    \# Do something useful again
    bb-flag-unset someotherstate
fi

**bb-flag-set** FLAG

Sets up `FLAG`.

**bb-flag-unset** FLAG

Removes `FLAG`.

**bb-flag-clean**

Removes all flags.

### read[¶](#read "Permanent link")

The module provides function to read [Java Properties](http://docs.oracle.com/javase/7/docs/api/java/util/Properties.html#load(java.io.Reader)), [INI](http://en.wikipedia.org/wiki/INI_file), [JSON](http://json.org/), and [YAML](http://www.yaml.org/) files into Bash variables. It uses [`bb-ext-python`](#bb-ext-python) to create read helpers. So you need Python to be installed to use this module.

Each reading function accepts optional `PREFIX` argument, which prepends result variable names. Any illegal char (which cannot be in the Bash variable name) will be replaced by `_` underscore one. So that keys like `dotted.key` will be imported as `dotted_key`.

Complex objects like hashes and arrays (from JSON and YAML) are unfolded to the flat variables. Nulls are treated as empty strings.

If the file doesn’t exist or cannot be read, the function logs error and returns `1`.

Each reading function has its helper one, which just prints variables to stdout. Such helper functions ends with `-helper` postfix. For examle, `bb-read-json-helper` is a helper for [`bb-read-json`](#bb-read-json). You can use these helpers for debugging.

**bb-read-properties** FILENAME \[PREFIX\]

The function reads [Java Properties](http://docs.oracle.com/javase/7/docs/api/java/util/Properties.html#load(java.io.Reader)) file `FILENAME` and parses it. The lines like `key=value` or `key: value` or even `key := value` are converted into Bash variables. For example, let `my.properties` file contains:

param1 = value1
param2 = long string

And the script can read it as the following:

bb-read-properties "my.properties" "conf\_"
echo "$conf\_param1"     \# prints "value1"
echo "$conf\_param2"     \# prints "long string"

If the same key appears multiple times, only the last value will be visible.

The escapes in the key name (like `k\:e\=y`) are *not supported*, the first `:` or `=` is treated as the end of the key name.

The multiline values (where the endline character is escaped by backslash) are *not supported* too.

**bb-read-ini** FILENAME \[SECTION \[PREFIX\]\]

The function reads [INI](http://en.wikipedia.org/wiki/INI_file) file `FILENAME` and parses it. The optional `SECTION` can be passed to read values from only this section. If `SECTION` is omitted or equals to `*`, all sections will be read. Each key will be prepended by its section name.

\[section\]
param \= value1

\[section:2\]
param \= long string

And the script can read it as the following:

bb-read-ini "my.ini" "\*" "conf\_"
echo "$conf\_section\_param"     \# prints "value1"
echo "$conf\_section\_2\_param"   \# prints "long string"

The function will use [SafeConfigParser](https://docs.python.org/2/library/configparser.html#ConfigParser.SafeConfigParser), if Python 2.x is default Python interpreter, or [ConfigParser](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser) for Python 3.x. See their documentation for details.

**bb-read-json** FILENAME \[PREFIX\]

The function reads [JSON](http://json.org/) file `FILENAME` and parses it. For example, let `my.json` file contains:

{
    "key": "value1",
    "object": { "key": "value2" },
    "array": \[1, { "key": "value3" } \]
}

And the script can read it as the following:

bb-read-json "my.json" "conf"  \# NOTE, there is no "\_" at the end of prefix
echo "$conf\_key"               \# prints "value1"
echo "$conf\_object\_key"        \# prints "value2"
echo "$conf\_array\_len"         \# prints "2", the length of array
echo "$conf\_array\_0"           \# prints "1", the first element of array
echo "$conf\_array\_1\_key"       \# prints "value3"

**bb-read-yaml** FILENAME \[PREFIX\]

The function reads [YAML](http://www.yaml.org/) file `FILENAME` and parses it. For example, let `my.yaml` file contains:

key: value1
object:
    key: value2
array:
    \- 1
    \- { "key": "value3" }

And the script can read it as the following:

bb-read-yaml "my.yaml" "conf"  \# NOTE, there is no "\_" at the end of prefix
echo "$conf\_key"               \# prints "value1"
echo "$conf\_object\_key"        \# prints "value2"
echo "$conf\_array\_len"         \# prints "2", the length of array
echo "$conf\_array\_0"           \# prints "1", the first element of array
echo "$conf\_array\_1\_key"       \# prints "value3"

The function depends on [PyYaml](http://pyyaml.org/), which is not in the Python standard library. Use [`bb-read-yaml?`](#bb-read-yaml_) function to check whether PyYaml is installed. For example:

bb-read-yaml? || pip install pyyaml

**bb-read-yaml?**

Checks whether [PyYaml](http://pyyaml.org/) is installed, so that function [`bb-read-yaml`](#bb-read-yaml) can be used.

### sync[¶](#sync "Permanent link")

The module provides functions for synchronization files and directories. The main goal is delaying [events](#event), if source and destination files are different. That is why it does not use [rsync](http://rsync.samba.org/) command.

**bb-sync-file** DST\_FILE SRC\_FILE \[EVENT \[ARGUMENTS...\]\]

Synchronizes contents of `DST_FILE` with `SRC_FILE`. If `DST_FILE` is changed it will [delay](#bb-event-delay) `EVENT` with `ARGUMENTS`. Usage:

bb-event-on restart-server "service nginx restart"

bb-sync-file "/etc/nginx/sites-available/default" "my\_site.conf" restart-server

Each time `my_site.conf` is changed, the script above will update Nginx configuration and restart it.

Additionally, if `DST_FILE` is changed, an event `bb-sync-file-changed` will be [fired](#bb-event-fire) with the file path as an argument.

**bb-sync-dir** \[OPTIONS\] DST\_DIR SRC\_DIR \[EVENT \[ARGUMENTS...\]\]

Synchronizes contents of `DST_DIR` with `SRC_DIR`. If `DST_DIR` is changed it will [delay](#bb-event-delay) `EVENT` with `ARGUMENTS`.

**Options:**

*   `-o` `--one-way` perform one-way synchronization. This means that all files in `SRC_DIR` will be replicated to `DST_DIR`, but files from `DST_DIR` that are not in `SRC_DIR` **will not be affected**.

*   `-t` `--two-way` perform two-way synchronization. This means that all files in `SRC_DIR` will be replicated to `DST_DIR`, and files from `DST_DIR` that are not in `SRC_DIR` **will be removed**. This is **default** behavior.

Additionally for each file or directory affected by synchronization it will [fire](#bb-event-fire) the following events with the full path to the file/directory passed as an argument:

*   `bb-sync-file-changed`
*   `bb-sync-file-removed`
*   `bb-sync-dir-created`
*   `bb-sync-dir-removed`

### wait[¶](#wait "Permanent link")

**bb-wait** CONDITION \[TIMEOUT\]

Freezes scripts until `CONDITION` is evaluated as `true`, i.e. expression returns non-zero status code. Example:

LOG\="$( bb-tmp-file )"
start-some-server 2\> "$LOG"
bb-wait 'cat "$LOG" | grep "Server ready"'
\# Do something useful using that server

If the optional `TIMEOUT` is not passed, the function will wait for `CONDITION` forever. If `TIMEOUT` has been specified and reached during the command execution, it will logs error and return `1`.

### iptables[¶](#iptables "Permanent link")

Manage iptables chains & rules, playing nice with existing rules. To reliably identify rules, an "ID" is required for each rule. It must be unique to the chain. Use whatever convention is preferred for ID's. Simple string matching is used.

**bb-iptables-chain** CHAIN

Create chain `CHAIN` if does not exist. Example:

bb-iptables-chain WEB

**bb-iptables-rule** -t,--table TABLE=filter -n,--num NUM=-1 CHAIN ID

Define rule in `CHAIN` in `TABLE` at position `NUM`. If rule with matching `ID` exists, then update it. When `NUM` is negative, count from end of `CHAIN` (-1 == last rule).

\# Add just before the end (-2), useful when last rule defines the policy.
bb-iptables-rule --num -2 INPUT https -p tcp --dport 443 -j WEB
\# Append some rules to end of chain WEB of the filter table.
bb-iptables-chain WEB
bb-iptables-rule WEB host-a --src $HOST\_A -j ACCEPT
bb-iptables-rule WEB host-b --src $HOST\_B -j ACCEPT
\# Insert at beginning (1) of the nat table.
bb-iptables-rule INPUT 'container subnet' -2 -j ACCEPT -s $NET
bb-iptables-rule --num 1 --table nat POSTROUTING 'container internet access' 1 -j MASQUERADE -s $net ! -o docker0
bb-iptables-rule --num 1 --table nat POSTROUTING tcp:1 -p tcp --dport 1 -j ACCEPT

### task[¶](#task "Permanent link")

The module provides functions to define and run tasks. Each task can define its dependencies (other tasks), that will run within it. Each task will be executed only once within the call of [`bb-task-run`](#bb-task-run), even if it is included by several tasks as dependency. If any of task exits with non-zero code (i.e. fails), [`bb-exit`](#bb-exit) function will be called with the same code.

Example:

bb-task-def 'install-build-tools'
install-build-tools() {
    \# ...
}

bb-task-def 'build-frontend'
build-frontend() {
    bb-task-depends 'install-build-tools'
}

bb-task-def 'build-backend'
build-backend() {
    bb-task-depends 'install-build-tools'
}

bb-task-def 'build-app'
build-app() {
    bb-task-depends 'build-backend' 'build-frontend'
}

\# The following code will execute tasks:
\# \* install-build-tools (only once)
\# \* build-backend
\# \* build-frontend
\# \* build-app
bb-task-run 'build-app'

**bb-task-def** TASK\_NAME \[FUNC\_NAME\]

Defines task `TASK_NAME` as function `FUNC_NAME`. If `FUNC_NAME` is omitted, `TASK_NAME` will be used instead. Example:

bb-task-def 'test' 'run-test-suite'
run-test-suite() {
    \# Function name differ from task name to avoid conflict with
    \# built-in \`test\` function.
}

**bb-task-depends** TASK \[TASK...\]

Runs specified tasks within the current task. This function can be called only within another task.

**bb-task-run** TASK \[TASK...\]

Runs specified tasks.

### apt[¶](#apt "Permanent link")

The module provides functions to work with [Apt](https://wiki.debian.org/Apt) package manager.

**bb-apt?**

Checks if Apt is available. Usage:

if bb-apt?
then
    bb-apt-install somepackage
fi

**bb-apt-repo?** REPOSITORY

Checks if `REPOSITORY` is installed. Usage:

REPO\='http://example.com/repo/ubuntu/'
if bb-apt-repo? $REPO
then
    cp /etc/apt/sources.list /etc/apt/sources.list.backup
    echo "deb $REPO precise main" >> /etc/apt/sources.list
    echo "deb-src $REPO precise main" >> /etc/apt/sources.list
fi

**bb-apt-package?** PACKAGE

Checks if `PACKAGE` is installed.

**bb-apt-update**

Updates Apt cache. It sets up variable `BB_APT_UPDATED` to `true`. So the second call of this function does nothing.

**bb-apt-install** PACKAGE \[PACKAGE...\]

Installs `PACKAGE` if it is not already installed. It uses [`bb-apt-package?`](#bb-apt-package) for checking `PACKAGE` installation status, and [`bb-apt-update`](#bb-apt-update) for updating Apt cache before installation.

For each installed package an event `bb-package-installed` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some post installation actions. For instance, install [MySQL on Ubuntu without asking a password](http://stackoverflow.com/a/7740393/3182064):

bb-event-on 'bb-package-installed' 'post-install'
post-install() {
    local PACKAGE\="$1"
    case "$PACKAGE" in
        "mysql-server")
            \# Setup MySQL root password
            mysqladmin -u root password 'myRooT\_pa$$w0rd'
            ;;
    esac
}
\# Do not ask for MySQL root password during installation
export DEBIAN\_FRONTEND\=noninteractive
bb-apt-install mysql-server

If package is unable to be installed, script will be terminated with error, i.e. [`bb-exit`](#bb-exit) will be called.

**bb-apt-package-upgrade?** PACKAGE

Checks if a new version of `PACKAGE` is available. It uses [`bb-apt-update`](#bb-apt-update) for updating Apt cache before doing the check.

If the requested package is not installed, `false` is returned by the function.

**bb-apt-upgrade** PACKAGE \[PACKAGE...\]

Upgrades `PACKAGE` if a newer version is available. It uses [`bb-apt-package-upgrade?`](#bb-apt-package-upgrade) for checking the availability of an updated version.

Before upgrading a package, an event `bb-package-pre-upgrade` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some pre upgrade actions.

After upgrading a package, an event `bb-package-post-upgrade` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some post upgrade actions.

If package is unable to be upgraded, script will be terminated with error, i.e. [`bb-exit`](#bb-exit) will be called.

### yum[¶](#yum "Permanent link")

The module provides functions to work with [Yum](http://yum.baseurl.org/) package manager.

**bb-yum?**

Checks if Yum is available. Usage:

if bb-yum?
then
    bb-yum-install somepackage
fi

**bb-yum-repo?** REPOSITORY

Checks if `REPOSITORY` repository is installed. Usage:

if bb-yum-repo? somerepo
then
    rpm -ivh "http://example.com/repo/centos/somerepo.noarch.rpm"
fi

**bb-yum-package?** PACKAGE

Checks if `PACKAGE` is installed.

**bb-yum-update**

Updates Yum cache. It sets up variable `BB_YUM_UPDATED` to `true`. So the second call of this function does nothing.

**bb-yum-install** PACKAGE \[PACKAGE...\]

Installs `PACKAGE` if it is not already installed. It uses [`bb-yum-package?`](#bb-yum-package) for checking `PACKAGE` installation status, and [`bb-yum-update`](#bb-yum-update) for updating Yum cache before installation.

For each installed package an event `bb-package-installed` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some post installation actions. For instance, [setup PostgreSQL on CentOS](http://www.postgresql.org/download/linux/redhat/):

bb-event-on 'bb-package-installed' 'post-install'
post-install() {
    local PACKAGE\="$1"
    case "$PACKAGE" in
        "postgresql-9.3")
            chkconfig postgresql-9.3 on
            service postgresql-9.3 initdb
            service postgresql-9.3 start
            ;;
    esac
}
bb-yum-install postgresql93-server

If package is unable to be installed, script will be terminated with error, i.e. [`bb-exit`](#bb-exit) will be called.

### brew[¶](#brew "Permanent link")

The module provides functions to work with [Homebrew](http://brew.sh/) package manager.

**bb-brew?**

Checks if Homebrew is available. Usage:

if bb-brew?
then
    bb-brew-install somepackage
fi

**bb-brew-repo?** REPOSITORY

Checks if `REPOSITORY` repository (tap in Homebrew terms) is installed.

**bb-brew-package?** PACKAGE

Checks if `PACKAGE` is installed.

**bb-brew-cask-package?** PACKAGE

Checks if `PACKAGE` is installed.

**bb-brew-update**

Updates Homebrew cache. It sets up variable `BB_BREW_UPDATED` to `true`. So the second call of this function does nothing.

**bb-brew-install** PACKAGE \[PACKAGE...\]

Installs `PACKAGE` if it is not already installed. It uses [`bb-brew-package?`](#bb-brew-package) for checking `PACKAGE` installation status, and [`bb-brew-update`](#bb-brew-update) for updating Homebrew cache before installation.

For each installed package an event `bb-package-installed` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some post installation actions.

If package is unable to be installed, script will be terminated with error, i.e. [`bb-exit`](#bb-exit) will be called.

**bb-brew-cask-install** PACKAGE \[PACKAGE...\]

Installs `PACKAGE` if it is not already installed. It uses [`bb-brew-cask-package?`](#bb-brew-cask-package) for checking `PACKAGE` installation status, and [`bb-brew-update`](#bb-brew-update) for updating Homebrew cache before installation.

For each installed package an event `bb-package-installed` will be fired by [`bb-event-fire`](#bb-event-fire) with the package name as an argument. So that you will be able to make some post installation actions.

If package is unable to be installed, script will be terminated with error, i.e. [`bb-exit`](#bb-exit) will be called.

### augeas[¶](#augeas "Permanent link")

The module provides functions to work with [Augeas](http://augeas.net) configuration editing tool.

**BB\_AUGEAS\_EXTRA\_COMMANDS**

The variable stores extra Augeas commands that will be run before the ones embeded in bb-augeas functions.

By default, the variable is empty and no additionnal command is provided.

**bb-augeas?**

Checks if Augeas is available.

**bb-augeas-get** FILE SETTING

Gets the value of `SETTING` from file `FILE`. Usage:

VALUE\="$(bb-augeas-get "/etc/ssh/sshd\_config" "Port")"
if bb-error?
then
    \# Handle read error
else
    \# Do something useful
    echo "Configured SSH port is $VALUE"

**bb-augeas-set** FILE SETTING VALUE \[EVENT \[ARGUMENTS...\]\]

Sets the value of `SETTING` to `VALUE` in file `FILE`. If `FILE` is changed, it will [delay](#bb-event-delay) `EVENT` with `ARGUMENTS`. Usage:

bb-event-on restart-server "service ssh restart"

bb-augeas-set "/etc/ssh/sshd\_config" "Port" "22" restart-server

Also, if `FILE` is changed, an event `bb-augeas-file-changed` will be fired by [`bb-event-fire`](#bb-event-fire) with the file path as an argument. So that you will be able to make some file-specific actions.

**bb-augeas-match?** FILE SETTING VALUE

Checks if the value of `SETTING` in file `FILE` matches the value `VALUE`.

if bb-augeas-match? "/etc/ssh/sshd\_config" "Port" "22"
then
    \# Do something useful
fi

## Task Runner[¶](#task-runner "Permanent link")

It is an experimental feature. You can find `install.sh` script at `build` directory of the sources or within distributive archive. This script installs Bash Booster to your system (should be run with root privileges, of course) with task runner utility `bb-task`.

Usage is quite simple and is similar to [Make](https://www.gnu.org/software/make/) and other Make-like tools. Place file `bb-tasks.sh` into your project directory with task definitions (see [`task`](#task) module). And run tasks using:

$ bb-task task-name

This command will:

*   read Bash Booster configuration from `/etc/bashbooster/bbrc`, `~/.bbrc` (home directory), and `./.bbrc` (current directory);
*   initialize Bash Booster itself;
*   read your task definitions from `./bb-tasks.sh`;
*   run specified tasks.

See `examples/task-runner` at the sources for live demo.

## Support & Feedback[¶](#support-feedback "Permanent link")

Visit our [discussion group](https://groups.google.com/forum/#!forum/bash-booster) if any support is required. It is a good place for proposals too. And of course, any feedback will be highly appreciated, either good and bad.

## Contribution[¶](#contribution "Permanent link")

Bug reports and pull requests are welcome on [BitBucket](https://bitbucket.org/kr41/bash-booster/).

The source code is covered by unit tests where it is possible. If you are going to add some new features, try to keep them covered too. Use the following command to run tests:

$ ./test.sh

Tests themselves are placed into `unit tests` directory. Yes, with the space char in the name. It helps to catch stupid errors with unquoted variables.

## License[¶](#license "Permanent link")

The code is licensed under the terms of GNU GPL version 3 license. The full text of the license can be found at the root of the sources or at [GNU website](http://www.gnu.org/licenses/licenses.html).

## Changes[¶](#changes "Permanent link")

### 0.6 (2019-02-21)[¶](#06-2019-02-21 "Permanent link")

*   Fixed MacOS compatibility
*   Added `brew cask` functions (Oliver Marshall)

### 0.5beta (2017-12-14)[¶](#05beta-2017-12-14 "Permanent link")

*   Fixed `bb-yum-repo?` funcion

### 0.4beta (2016-09-20)[¶](#04beta-2016-09-20 "Permanent link")

*   Added `iptables` module (Erik Stephens)
*   Added `augeas` module (Jocelyn Le Sage)
*   Added some shortcuts to `assert` module (Jocelyn Le Sage)
*   Added support of `upgrade` comand to `apt` module (Jocelyn Le Sage)
*   Fixed and improved `sync` module (Jocelyn Le Sage)
*   Fixed `bb-event-delay` function (Jocelyn Le Sage)
*   Fixed `task` module

### 0.3beta (2015-03-07)[¶](#03beta-2015-03-07 "Permanent link")

*   Added `task` module and task runner utility
*   Added `brew` module (Trevor Bekolay)
*   Added helper functions `bb-error?` and `bb-exit-on-error`
*   Updated `download` module to be more error-proof and flexible
*   Fixed `bb-template` function
*   Fixed cleanup process
*   Fixed behavior on shared workspace. Several scripts can now use single workspace directory at the same time.

### 0.2beta (2014-10-11)[¶](#02beta-2014-10-11 "Permanent link")

*   Changed license to GNU GPL version 3
*   Added `bb-log-deprecated` function to `log` module
*   Added `assert` module
*   Added `ext` module
*   Added `exe` module
*   Added `read` module
*   Added `wait` module
*   Marked `properites` module as deprecated in favor of `read` one
*   Rewrote unit tests

### 0.1beta5 (2014-10-10)[¶](#01beta5-2014-10-10 "Permanent link")

*   Fixed #3 OS X support (Mike Kolganov)
*   Fixed #1 Ubuntu 14.04 support

### 0.1beta4 (2014-09-08)[¶](#01beta4-2014-09-08 "Permanent link")

*   Added `properties` module (Denis Nelubin)

### 0.1beta3 (2014-08-20)[¶](#01beta3-2014-08-20 "Permanent link")

*   When package is installed by function `bb-apt-install` or `bb-yum-install`, an event `bb-package-installed` will be fired with the package name as an argument. So that you will be able to make some post installation actions.
*   If package is unable to be installed, `bb-apt-install` or `bb-yum-install`, will terminate script with error.

### 0.1beta2 (2014-07-23)[¶](#01beta2-2014-07-23 "Permanent link")

*   Added ability to pass arguments to event handlers

### 0.1beta (2014-07-16)[¶](#01beta-2014-07-16 "Permanent link")

*   Intial release
