# MacClam

The Non-Graphical ClamAV Antivirus Solution for Mac OS X

I wrote this as a free alternative to the excellent <a
href="https://www.clamxav.com/" target="_blank">ClamXav</a>.  MacClam
sets up real-time directory monitoring and schedules periodic scans.
It uses <a href="http://www.clamav.net/" target="_blank">ClamAV</a> as
an AntiVirus engine and <a
href="https://github.com/emcrisostomo/fswatch"
target="_blank">fswatch</a> to actively monitor directories for new or
changed files, which are then sent to clamd for scanning.  Periodic
full scans are scheduled with cron.  It also provides a way to scan
individual files or directories on demand from the command line.

I have tested MacClam on Mojave (macOS 10.14).  but it may also work
in other versions of macOS.

## Prerequisites ##

You will need to have Apple's <a
href="https://developer.apple.com/library/ios/technotes/tn2339/_index.html">Xcode
command line tools</a> which can be installed with

    xcode-select --install

Then click "Install". After you have installed the command line tools,
if you are on Mojave, you will also need to install the macOS headers
package with

    sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /

## Installation ##

Installation is very simple.  After installing prerequisite tools, type the following in a terminal.

    curl -O https://raw.githubusercontent.com/killdash9/MacClam/master/MacClam.sh
    chmod +x MacClam.sh
    ./MacClam.sh

This will bootstrap MacClam by building the lastest versions of ClamAV
and fswatch from source.  It will schedule a full file system scan
once a week and update signatures once a day.  It also sets up live
monitoring for the $HOME and /Applications directories.  Each of these
things can be configured by modifying script variables and re-running
the script.

By default, the installation directory is `~/MacClam`.  This directory
contains all the source, binaries, log files, and quarantine folder.
The only artifact of the installation outside this directory is the
crontab and the `MacClam.sh` script itself, which is required for
MacClam to function.  If you want to move the `MacClam.sh` script to
another location, just re-run it from the new location and the crontab
references will be updated.  It can be totally uninstalled by
running `./MacClam.sh uninstall`.

## Usage ##

`./MacClam.sh` does the following:

* Builds clamd and fswatch from source if needed
* Sets up regular signature updates and full scans in crontab
* Updates clamd signatures
* Starts active monitoring services clamd and fswatch if not already running
* Sets active monitoring to run on startup (also done in crontab)
* If run from a terminal, it will show any current scanning activity

The following command

    ./MacClam.sh /path/file_or_directory ...

does everything previously listed, and then runs clamscan on the files
or directories.  Multiple files or directories can be specified.

    ./MacClam.sh quarantine

Opens the quarantine folder in Finder.  By default, this is
`~/MacClam/quarantine`

    ./MacClam.sh help
    
Displays a help message.

    ./MacClam.sh uninstall

Uninstalls MacClam.  More specifically, it stops clamd and fswatch.
It removes MacClam entries from the crontab.  It moves the quarantine
directory from the MacClam installation directory to
~/MacClam_quarantine, just in case there's something in there you
want.  It deletes the MacClam installation directory which contains
clamav and fswatch.  It does not delete the MacClam.sh file, and you
can reinstall MacClam by running it again.

## Customization ##

Scheduled scans, monitoring and installation location can be
configured by editing configuration variables at the beginning of the
script, and then running the script again to apply your changes.

## Design Principle ##

MacClam.sh is designed to have a very simple interface -- one command
to do everything.  It is idempotent, meaning that re-running
MacClam.sh will do nothing if everything is set up correctly and
services are running.  If there are changes in the configuration
variables, it will make sure they are applied, and restart services as
necessary.

## Virus Scans ##

MacClam performs three types of scans:

1. Active monitoring: MacClam will monitor any directories you specify
   for activity.  When a file is changed or created, it will be
   scanned immediately.  By default, the $HOME and Applications
   directories are monitored.
2. Scheduled scanning: MacClam will perform recursive scans of
   directories at scheduled times.  By default, the entire hard drive
   is scanned once a week.  Scheduling is done with cron.
3. On-demand scanning: Running `MacClam.sh` with one or more file or
   directory arguments will scan the files or directories specified.

In all cases, when a virus is found, it is moved to the quarantine
folder.  For active monitoring, when a virus is identified, a brief
graphical notification is shown in the top-right corner.
