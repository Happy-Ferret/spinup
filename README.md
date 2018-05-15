# spinup

Spinup is a simple process runner for use during development, designed for bringing up all necessary servers and file-watchers with a single command.

Using Spinup is as simple as creating a `spin.up` file containing one shell command per line in your project's root directory and then running the `spinup` executable.

![spinup demo](demo.gif)

Spinup is pretty customisable. Let's take a look at an example config file:

```
# (lines beginning with # are comments...)

# Directives are denoted by "!" and come before any commands
# Directives are used to configure global settings.
# (!prefix sets the prefix for each line of output)
!prefix [%p]

# Run the webserver
python -m SimpleHTTPServer 9000

# Compile some JavaScript
watchify -o bundle.js main.js

# Per-command options are prefixed with @ and will be applied to the next command
# (@cd sets the working directory for the next command)
@cd www
php -S 127.0.0.1:8000

# Shell arguments are parsed correctly...
echo "let's test" "the argument" parser

# ...and environment variables can be used too:
echo "your home directory is:" $HOME
```

`spinup` will run until all child processes have exited. Hit `Ctrl-C` to send `SIGTERM` to any that are still running (this kill signal can be customised on a per-command basis, see the `@kill` option below).

`dotenv` is also supported; any environment variables defined within `.env` will made available to the commands listed in `spin.up`.

## Installation

Spinup can be installed globally, or run locally via `npx`. To install globally:

```shell
$ npm install -g spinup
```

For local usage:

```shell
$ npm install spinup
$ touch spin.up
$ npx spinup
```

## Usage

```
$ spinup [-g | --group $group_list] [config]
```

  - `-g | --group`: optional comma separated list of groups to run; can be specified multiple times
  - `config`: optional path to configuration file, defaults to `./spin.up`.

## Directives

Leading lines of the `spin.up` file can include _directives_. Directives begin with a `!`, followed by the name of the directive and then its arguments.

### `!default`

Sets an environment variable if it does not already exist.

```
!default PORT 3000
echo $PORT
```

```shell
$ spinup
3000
$ PORT=5000 spinup
5000
```

See also: `!set`, to unconditionally set an environment variable.

### `!noprefix`

Do not add a prefix onto each line of output.

See also: `!prefix`.

### `!ports`

The `!ports` directive can be used to automatically create any number of environment variables with sequential integer values, starting from a given base. This is useful for generating port numbers for your processes to listen on, connect to etc. Because the values are written to environment variables its possible to refer to the same value multiple times, i.e. in instances where one process needs to communicate with another. Let's take a look at how it works.

This first example generates three port numbers, `$A`, `$B` and `$C`, starting from 5000. So `$A` is 5000, `$B` is 5001 etc:

```
!ports 5000 $A $B $C
```

It's also possible to specify an optional override variable. If this environment variable is present its value will be used instead of the base port number:

```
!ports $BASE:9000 $WEB_SERVER $TILE_SERVER
```

In the above example, `$WEB_SERVER` and `$TILE_SERVER` will default to 9000 &amp; 9001, but this can be overridden by defining the environment variable `$BASE` to some other port number before invoking `spinup`.

Port numbers generated by the `!ports` directive are used in the same way as any other environment variable. Here's an example that spawns two mutually communicating processes:

```
!ports $BASE:8000 $P1 $P2

process1 --listen $P1 --connect $P2
process2 --listen $P2 --connect $P1
```

### `!prefix`

Sets the prefix to prepend to each line of output.

Supported substitutions:

  * `%t`: task number (i.e. the `t`th task listed in `spin.up`, starting from zero)
  * `%p`: process ID
  * `%c`: command; can optionally be suffixed with a number to restrict/pad output length
  * `%n`: command name, as specified by per-command option @name; if unspecified, defaults to command
  * `%Y`: year (4 digits)
  * `%y`: year (2 digits)
  * `%m`: month
  * `%d`: day
  * `%H`: hour
  * `%M`: minutes
  * `%S`: seconds

The default prefix is `[%t:%c6]`.

### `!set`

Sets an environment variable; will be available to all child processes.

```
!set PORT 3000
!set HOST 192.168.1.123:$PORT
```

See also: `!default`

## Per-command Options

Lines beginning with `@` define per-command options and will be applied to the next command read from the configuration file.

### `@cd DIRECTORY`

Set the working directory for the command.

### `@group GROUP_LIST`, `@groups GROUP_LIST`

Space-separated list of groups that the next command belongs to. The command line option `-g` / `--group` can be used to restrict which groups' commands are launched.

Any command for which `group` is not specified will belong to the `default` group.

### `@kill SIGNAL`

Set the name of the signal that should be used to kill the process.

### `@name NAME`

Set the name that should be displayed by the `%n` prefix option.

### `@noerror`

Do not treat the process' `stderr` as error output (usually highlighted in red), but instead use the process' specific color.

## Copyright &amp; License

&copy; 2014-2018 Jason Frame [ [@jaz303](http://twitter.com/jaz303) / [jason@onehackoranother.com](mailto:jason@onehackoranother.com) ]

Released under the ISC license.