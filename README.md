![gqrx-scanner](media/gqrx-scanner-repository.png)
# gqrx-scanner
[![CI](https://github.com/neural75/gqrx-scanner/actions/workflows/ci.yml/badge.svg)](https://github.com/neural75/gqrx-scanner/actions/workflows/ci.yml)

A frequency scanner for [Gqrx Software Defined Radio](http://gqrx.dk/) receiver

## Description

gqrx-scanner is a frequency scanner written in C that uses [gqrx remote protocol](http://gqrx.dk/doc/remote-control) to perform a fast scan of the band. It can be used in conjunction with the gqrx bookmarks (--mode bookmark) to look for the already stored frequencies or, in a free sweep scan mode (--mode sweep), to explore the band within a specified frequency range (--min, --max options).

The sweep scan uses an adaptive algorithm to remember the active frequencies encountered during the sweep, that prioritizes active stations without stopping to look for new ones.

Gqrx Squelch level is used as the threshold, when the signal is strong enough it stops the scanner on the frequency found.
After the signal is lost the scanner waits a configurable ammount of time and restart the loop (--delay option).

In sweep mode the scan of the band is performed fast (well, as fast as it can), self asjusting the scanning speed. On signal detection a fine tuning is performed to pinpoint the nearest carrier frequency (subdivision of upper and lower limits with increasing precision) and, for the already seen carriers, a previous value is used to avoid fine tuning at every hit. This value is also averaged out with the new one in order to converge eventually to the exact frequency in less time (after 4 hits of the same frequency).

## Features
* Support for Gqrx bookmarks file
* Fast sweep scan with adaptive monitor of the most active stations
* Frequency range constrained scan (also for bookmarks)
* Multiple Tag based search in bookmark scan mode ("Tag1|Tag2|TagX").
* Automatic Frequency Locking in sweep scan mode
* Interactive monitor to skip, ban or pause a frequency manually
* Automatic recording of detected signals

## Pre-requisites
Gqrx Remote Protocol must be enabled: Tools->Remote Control. See [this](http://gqrx.dk/doc/remote-control).

## Notes on Gqrx settings
It is advisable to disable AGC during the scan: adjust the fixed gain lowering the noise floor to at least -60/-70 dBFS and set the squelch level to -50/-40 dBFS, depending on the band activities and noise levels.

The best results are obtained in relative quiet frequencies with sporadic transmissions. If the band is cluttered with harmonics and other types of persistent noise, avoid the sweep scan and use the bookmarks mode with a higher squelch level (or use the 'b' key to ban a frequency from the scan).

In sweep mode use a limited bandwidth of about 2 MHz in order to avoid VFO and noise floor levels chainging during the sweep.
If you don't provide the --min, --max option, set the demodulator frequency to the middle of the screen and start from there in order to avoid the panadapter to move during the scan.

You may also consider to adjust FFT options: FFT size and rate (on FFT Settings) to improve performances (and cpu usage).
I have found better result with high fft size (64536) and 17 fps refresh rate, but this depends on your hardware.

## Command line Options
```
Usage:
gqrx-scanner
		[-h|--host <host>] [-p|--port <port>] [-m|--mode <sweep|bookmark>]
		[-f <central frequency>] [-b|--min <from freq>] [-e|--max <to freq>]
		[-R|--range <freq range>] [-n|--repeat <N>] [-S|--set-squelch <dB>]
		[-A|--save-active <file>] [-E|--exclude-active <file>]
		[-d|--delay <time>] [-l|--max-listen <time>]
		[-t|--tags <"tag1|tag2|...">]
		[-v|--verbose]
		[-r|--record]

-h, --host <host>            Name of the host to connect. Default: localhost
-p, --port <port>            The number of the port to connect. Default: 7356
-m, --mode <mode>            Scan mode to be used. Default: sweep
                               Possible values for <mode>: sweep, bookmark
-f, --freq <freq>            Frequency to scan with a range of +- 1MHz (or use --range).
                               Supports K/KHz and M/MHz suffixes (e.g., 156M, 144.5MHz, 500K)
                               Unidirectional syntax: -f 156M+1M (scan 156-157MHz upward)
                                                      -f 157M-2M (scan 155-157MHz downward)
                               Without suffix: must be >= 1000000 Hz
                               Default: the current frequency tuned in Gqrx. Incompatible with -b, -e
-b, --min <freq>             Frequency range begins with this <freq>. Incompatible with -f
                               Supports K/KHz and M/MHz suffixes or Hz (>= 1000000)
                               Examples: --min 147M, --min 147.5MHz, --min 147000000
-e, --max <freq>             Frequency range ends with this <freq>. Incompatible with -f
                               Supports K/KHz and M/MHz suffixes or Hz (>= 1000000)
                               Examples: --max 148M, --max 148.5MHz, --max 148000000
-s, --step <freq>            Frequency step. Default: 2500 (2.5KHz)
                               Supports K/KHz and M/MHz suffixes
                               Examples: --step 25K, --step 2.5K, --step 2500
-d, --delay <time>           Lingering time before scanner reactivates. Default: 2s (2000ms)
                               Supports: s (seconds), m (minutes), or milliseconds
                               Examples: --delay 5s, --delay 1m, --delay 2500
-l, --max-listen <time>      Maximum time to listen to active frequency. Default: 0 (no limit)
                               Supports: s (seconds), m (minutes), or milliseconds
                               Examples: --max-listen 10s, --max-listen 1m, --max-listen 5000
-x, --speed <time>           Bookmark scan speed. Default: 250ms
                               Supports: s (seconds), m (minutes), or milliseconds
                               If scan lands on wrong bookmark, use -x 500 or -x 1s to slow down
-R, --range <freq>           Frequency range for -f option. Default: 1MHz (1000000 Hz)
                               Supports K/KHz and M/MHz suffixes
                               Examples: --range 2M, --range 500K, --range 2000000
                               Cannot be used with -f <freq>+/-<offset> syntax
-n, --repeat <N>             Number of scan passes. Default: infinite
                               0 = one pass, N = N passes, omit for infinite scanning
                               Program exits after completing the specified number of passes
-S, --set-squelch <dB>       Set initial squelch level in Gqrx before scanning
                               Example: --set-squelch -50.5
-A, --save-active <file>     Save active frequencies to file (append mode)
                               Saves frequency after fine-tuning when signal is detected
                               Format: timestamp,freq_mhz,signal_level,squelch_level
                               Example: --save-active active_freqs.csv
-E, --exclude-active <file>  Exclude frequencies from listening (carriers, unwanted signals)
                               File format: one frequency per line (MHz or Hz)
                               Comments start with #
                               Example: --exclude-active excluded.txt
-y  --date                   Date Format, default is 0.
                               0 = mm-dd-yy
                               1 = dd-mm-yy
-q, --squelch_delta <dB>|a<dB> If set creates bottom squelch just for listening.
                             It may reduce unnecessary squelch audio supress.
                             Default: 0.0
                             Ex.: 6.5
                             Place "a" switch before <dB> value to turn into auto mode
                             It will determine squelch delta based on noise floor and
                             <dB> value will determine how far squelch delta will be placed from it.
                             Ex.: a0.5
-t, --tags <"tags">          Filter signals. Match only on frequencies marked with a tag found in "tags"
                               "tags" is a quoted string with a '|' list separator: Ex: "Tag1|Tag2"
                               tags are case insensitive and match also for partial string contained in a tag
                               Works only with -m bookmark scan mode
-r, --record                 Enable recording of detected signals
-v, --verbose                Output more information during scan (used for debug). Default: false
--help                       This help message.

```

## Interactive Commands
These keyboard shortcuts are available during scan:
```
[space] OR [enter]  :   Skips a locked frequency (listening to the next).
'b'                 :   Bans a locked frequency, the bandwidth banned is about 10 Khz from the locked freq.
'c'                 :   Clears all banned frequencies.
'p'                 :   Pauses scan on locked frequency, 'p' again to unpause.
```

## Examples
Performs a sweep scan with a range of +-1Mhz from the demodulator frequency in Gqrx:
```
./gqrx-scanner
```
<br>

Search all bookmarks for frequencies in the range +- 1 Mhz from the demodulator frequency in Gqrx:
```
./gqrx-scanner -m bookmark
```
<br>

Performs a sweep scan from the central frequency 144.000 MHz using the range 143.000-145.000 MHz:
```
./gqrx-scanner -f 144000000
```
<br>

Performs a scan using Gqrx bookmarks, monitoring only the frequencies tagged with "DMR" or "Radio Links" in the range 430MHz-431MHz:

```
./gqrx-scanner -m bookmark --min 430M --max 431M --tags "DMR|Radio Links"
```
<br>

Performs a sweep scan from frequency 430MHz to 431MHz, using a delay of 3 seconds as idle time after a signal is lost:
```
./gqrx-scanner --min 430M --max 431M -d 3s
```
<br>

**New frequency suffix support examples:**

Scan 156-157 MHz using simplified notation with 25KHz steps:
```
./gqrx-scanner -f 156M --range 500K --step 25K
```
<br>

Scan from 155MHz to 157MHz (upward only) once and exit:
```
./gqrx-scanner -f 155M+2M --repeat 0
```
<br>

Scan 440-442MHz with custom squelch, listening max 10 seconds per signal:
```
./gqrx-scanner -f 441M --range 1M --set-squelch -55 --max-listen 10s
```
<br>

Scan 144-145MHz exactly 5 times with 2.5KHz steps:
```
./gqrx-scanner -f 144.5M --range 500K --step 2.5K --repeat 5
```
<br>

**Active frequency logging and exclusion examples:**

Scan 440-441MHz and save all detected active frequencies to CSV file:
```
./gqrx-scanner -f 440M --range 500K --save-active active_freqs.csv
```
<br>

Scan while excluding known carrier frequencies and unwanted signals:
```
./gqrx-scanner -f 440M --range 500K --exclude-active excluded.txt
```

Example `excluded.txt` file format:
```
# Excluded frequencies - carriers and unwanted signals
# One frequency per line (MHz or Hz)
440.500
441.200
# DMR carrier
440.125
```
<br>

Complete monitoring setup with logging and exclusions:
```
./gqrx-scanner -f 440M --range 1M \
  --exclude-active excluded.txt \
  --save-active active_freqs.csv \
  --max-listen 10s \
  --set-squelch -55
```

### Sample output

```
$ ./gqrx-scanner -m bookmark -f 430000000
Frequency range set from 429.000 MHz to 431.000 MHz.
[04-08-17 19:51:54] Freq: 430.037 MHz active [ Beigua                   ], Level: -40.20/-50.70  [elapsed time 03 sec]
[04-08-17 19:51:57] Freq: 430.288 MHz active [ ponte dmr                ], Level: -39.00/-50.70  [elapsed time 43 sec]
[04-08-17 19:52:40] Freq: 430.887 MHz active [ DMR                      ], Level: -30.50/-50.70  [elapsed time 11 sec]
[04-08-17 19:53:23] Freq: 430.900 MHz active [ Genova DMR               ], Level: -32.20/-50.70  [elapsed time 14 sec]
```

## Build and Install

```
cmake .
make
sudo make install
```

### Building and Running Tests

The test suite requires the cmocka testing framework. Install it first:

**Ubuntu/Debian:**
```
sudo apt-get install libcmocka-dev
```

**Fedora/RHEL:**
```
sudo dnf install libcmocka-devel
```

**macOS (via Homebrew):**
```
brew install cmocka
```

Once cmocka is installed, build and run tests:
```
cmake .
make
ctest
```

For verbose test output:
```
ctest --verbose
```

**Note:** If cmocka is not installed, the build will proceed normally but tests will be disabled. You'll see a message: "cmocka not found - tests disabled."

## To build for Mac-OSX

run `ccmake`, toggle, and set CMAKE_C_FLAGS=-DOSX

```
ccmake .
cmake .
make
sudo make install
```

## Uninstall
```
sudo make uninstall
```


## TODOs
* set modulation in bookmark search
* parsable output in csv?
