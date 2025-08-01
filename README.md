# run-at: schedule gcode execution

A series of shell scripts and gcode macros that enables scheduled gcode execution using the [`at` shell utility](https://www.redhat.com/en/blog/linux-at-command). Schedule lights to turn on at a certain time, or the bed to start heating up at a certain time, or even start a print at a scheduled time. 

**NOTE:** Running arbitrary gcode on your machine can lead to bad stuff. When scheduling gcode execution, make sure you know what the gcode is doing, and make sure the machine is safe to execute the gcode at the scheduled time. Use this at your own risk!

## Installation

These steps are to be done on your Pi (ssh in, use a monitor, whatever)

1. Install the `at` utility:

    ```
    sudo apt install at
    ```

1. Install the [G-Code Shell Command Extension](https://github.com/dw-0/kiauh/blob/master/docs/gcode_shell_command.md)

    **NOTE:** Read the warnings at the above link carefully. This extension allows remote execution of arbitrary shell commands from Klipper, and in turn any Klippper interfaces and frontends you are using (e.g. Moonraker + Mainsail/Fluidd). Lots of abuse can happen if your Klipper host isn't secured properly!

    The easiest way to do this is to use [`kiauh`](https://github.com/dw-0/kiauh/tree/master) (Advanced > G-Code Shell Command)

    If you don't want to use `kaiuh`, copy [`gcode_shell_command.py`](https://github.com/dw-0/kiauh/blob/master/resources/gcode_shell_command.py) from the `kaiuh` repository to `klippy/extras/`. 

1. Clone this repository:

    ```
    git clone https://github.com/kaine119/klipper-run-at
    cd klipper-run-at
    ```

1. Make a `~/bin` folder, and copy the contents of this repo's `bin` folder in:

    ```
    mkdir -p ~/bin
    cp -r ./bin/* ~/bin/
    ```

1. Copy the contents of [`run-at.cfg`](https://github.com/kaine119/klipper-run-at/blob/master/run_at.cfg) to your printer config

## `at` timespecs

The `TIME` arguments for the following macros accept "timespecs" that describe when the task should be scheduled; these are directly passed to `at`. There's a good description of what a timespec looks like [here](https://pubs.opengroup.org/onlinepubs/9799919799/utilities/at.html#tag_20_05_05).

Examples:

* Exact times can be given in 24h or 12h formats: `15:24`, `3:24pm`
* If you need a particular date, it goes after the timestamp: `3:24pm Oct 31`
    * Two keywords also exist for `today` and `tomorrow`: `3:50pm tomorrow`
* Times can also be given relative to `now`: `now + 20 minutes`
    * Available units: `minutes`/`min`, `hours`, `days`, `weeks`, `months`, `years` (singulars also accepted)

## Usage

Any argument with spaces needs to be surrounded with quotes, e.g. `RUN_AT TIME="now + 10 min" GCODE="M118 hello"`

G-code commands include any available Klipper g-code invocations, including any custom-defined macros. (Essentially anything you can type into the Klipper console.)

* `RUN_AT TIME=<timespec> GCODE=<gcode to execute>`

    Execute GCODE at TIME. 

* `SCHEDULE_BED_TEMP TIME=<timespec> TEMP=<bed temp>`

    Sets the bed temperature to TEMP at TIME (with M140). Useful for scheduling a chamber pre-heat.

* `SELECT_LATEST_GCODE`

    [Selects](https://marlinfw.org/docs/gcode/M023.html) the last uploaded print gcode file for printing. Does not actually start the print immediately, but "queues" it up for printing later for `SCHEDULE_PRINT_START`.

* `SCHEDULE_PRINT_START TIME=<timespec>`

    [Prints the selected file](https://marlinfw.org/docs/gcode/M024.html) at TIME.

* `LIST_JOBS`

    Prints a list of jobs to the console, including a job number, the job's scheduled execution time, and the command executed. The actual gcode is the part between `echo` and `>`

    The job number can be used with `DELETE_JOB` to cancel a job.

* `DELETE_JOB JOB=<job number>`

    Delete a job from the schedule.

## Useful links

* [Klipper gcodes page](https://www.klipper3d.org/G-Codes.html)
* [Marlin gcode list for more common operations](https://www.klipper3d.org/G-Codes.html)
* [`at` timespec description](https://pubs.opengroup.org/onlinepubs/9799919799/utilities/at.html#tag_20_05_05)

## Gentle reminders

* Know what your gcode is doing before you schedule it. If in doubt, execute commands manually in a controlled environment before scheduling it.
* Make sure operating conditions are/will be safe when scheduled gcode executes. If you schedule a command that moves the toolhead, don't stick your hands into the machine when the command executes. Remember to delete jobs if it is no longer safe to execute them
* Secure your Pi / your network when you have `gcode_shell_command` installed. If an attacker can get to Mainsail, they will be able to run arbitrary shell commands on the Pi itself.