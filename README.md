# gr-satellites
GNUradio decoders for different satellites.

This repository is a collection of GNUradio decoders for the telemetry of
several satellites. The decoders don't need a graphical interface to run, so
they can be used in an embedded or remote computer. The decoders are designed to
run in real time and print the telemetry packets to the terminal as they are
received. Optionally, the telemetry packets can be uploaded in real time to the
[PE0SAT telemetry server](http://tlm.pe0sat.nl/) or any other telemetry server
that implements the SiDS (Simple Downlink Sharing Convention) protocol.

It is also possible to use the decoder with a recording (audio WAV or IQ file),
in case that the telemetry wasn't processed in real time. To do this, one has to
know the time and date at which the recording was started and the recording has
to be played back at normal speed. This allows the decoder to compute the
correct timestamps for the packets when uploading them to the telemetry
server. It also simplifies Doppler correction of the recording with Gpredict if
the recording was not Doppler corrected.

## Usage

The signal is fed to the decoders using a UDP stream. The format used is the
same that [gqrx](http://gqrx.dk/doc/streaming-audio-over-udp) uses. Therefore,
you can use gqrx to feed the signal to the decoders. You will have to set the
proper frequency, mode and bandpass in gqrx for the satellite you want to
receive. This is probably the easiest way to start using the decoders from
gr-satellites. Gqrx supports Doppler correction with Gpredict.

It is also possible to use the frontend streamers from
[gr-frontends](https://github.com/daniestevez/gr-frontends). This allow to
stream from different SDR hardware without using a GUI SDR program. It is
possible to perform Doppler correction with Gpredict. There are also frontend
streamers to use a conventional receiver connected via soundcard and recordings
(audio WAV and IQ).

Each satellite has its own decoder. You can open the `.grc` file with
`gnuradio-companion` and edit the parameters (they are on the upper part of the
flowgraph). You can also run the `.py` script and specify the parameters on the
command line. Use the -h flag to get help on how to specify the parameters. The
decoder will printing each telemetry packet in the terminal as soon as it
receives it.

## Satellites supported

  * `sat_3cat2`
    [3CAT-2](https://nanosatlab.upc.edu/en/missions-and-projects/3cat-2), which
    transmits 9k6 AX.25 BPSK telemetry in the 2m band. You must use wide SSB
    mode to receive this satellite.
  * `aausat_4`
    [AAUSAT-4](http://www.space.aau.dk/aausat4/), which transmits 2k4 or 9k6 GFSK
    telemetry in the 70cm band. It uses the CSP protocol and FEC with a r=2, k=7
    convolutional code and a (255,223) Reed-Solomon code. You must use FM mode
    to receive this satellite.
  * `gomx_3`
    [GOMX-3](https://directory.eoportal.org/web/eoportal/satellite-missions/g/gomx-3),
    which transmits 19k2 GFSK telemetry in the 70cm band. It uses the CSP
    protocol and fect with a (255,223) Reed-Solomon code. The beacons include
    information from ADS-B beacons transmitted by terrestrial aircraft. You must
    use FM mode to receive this satellite.

## Required GNUradio OOT modules

The following GNUradio out-of-tree modules are required in several of the
decoders. You should probably install all of them.

  * [gr-kiss](https://github.com/daniestevez/gr-kiss) Tools for AX.25 and KISS
  * [gr-synctags](https://github.com/daniestevez/gr-synctags) Tools for dealing
     with GNUradio synctags easily
  * [gr-csp](https://github.com/daniestevez/gr-csp) Tools for CSP protocol
  * [gr-sids](https://github.com/daniestevez/gr-sids) Telemetry submission using
    the SiDS protocol

You also need to install Phil Karn's KA9Q `libfec` for some of the satellites
that use Reed-Solomon or convolutional codes (other include their own
decoder). A fork that builds in modern linux systems can be found
[here](https://github.com/daniestevez/libfec).

The following GNUradio out-of-tree modules are only required for the decoder of
one particular satellite. You may install only the ones you're interested in.

  * [gr-ax100](https://github.com/daniestevez/gr-ax100) GOMX-3 decoder and
    telemetry parser
  * [gr-aausat](https://github.com/daniestevez/gr-aausat) AAUSAT-4 decoder and
    telemetry parser
  * [gr-3cat2](https://github.com/daniestevez/gr-3cat2) 3CAT-2 telemetry parser

## Installing GNUradio OOT modules

This is the usual procedure to build and install an OOT module:

```bash
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig
```

## Submitting telemetry

To sumbit telemetry to the [PE0SAT telemetry server](http://tlm.pe0sat.nl/) (or
another SiDS telemetry server), you have to specify your callsign and
coordinates. The callsign is specified using the `--callsign` parameter and the
latitude and longitude are specified using the `--latitude` and `--longitude`
parameters if you are using the `.py` script. If you are using the `.grc` file
with `gnuradio-companion`, you can set these parameters by editing the boxes on
the upper part of the flowgraph.

The format for the latitude and longitude is of the form `00.00000` or
`-00.00000`. The `-` means South (for latitude) or West (for longitude).

If you want to submit telemetry from a recording, you have to specify the UTC
date and time when the recording was started. This allows the decoder to compute
the proper timestamp for the packets. The format is `YYYY-MM-DD HH:MM:SS` and it
is specified using `--recstart` if using the `.py` script or with the parameter
box on the upper part of the flowgrah if using the `.grc` file with
`gnuradio-companion`.

It is also **very important** that the decoder and the recording streamer are
started simultaneously. This can be achieved by something like
```bash
gr-frontends/wav_48kHz.py -f recording.wav & \
gr-satellites/sat_3cat2.py --recstart="2016-01-01 00:00" --callsign=N0CALL --latitude=0.000 --longitude=0.000
```

## Hints for receiving different modes

### Wide SSB

Some modes (9k6 BPSK, for instance) need to be received using SSB mode, but the
bandwidth of the signal is larger than the usual 3kHz bandwidth of a
conventional SSB receiver. Therefore, an SDR receiver or a heavily modified
conventional SSB receiver is needed (a 9k6 BPKS signal is about 15kHz wide).

The decoders for satellites using these kind of *wide SSB* signals expect the
signal to be centred at an audio frequency of 12kHz. This means that you have to
dial in USB mode to a frequency 12kHz lower than the nominal frequency of the
satellite (+/- Doppler). If your SDR program allows this (gqrx does), the best
idea is to set an SSB audio filter from 0Hz to 24kHz and then tune the signal in
the middle of the passband. Alternatively, you can use the `--bfo` parameter if
using the `.py` file or edit the corresponding parameter in the `.grc` file to
use a frequency different from 12kHz.

If you are using the wide SSB receivers from
[gr-frontends](https://github.com/daniestevez/gr-frontends) you don't need to do
anything special, as these receivers already dial in USB mode to a frequency
12kHz than the nominal and use a 24kHz wide audio filter.

### Receiving FSK and sideband inversion

We are all used to the two SSB modes: USB (which is sideband-preserving) and LSB
(which is sideband-inverting). When receiving FM (or FSK), there is the same
concept. An FM receiver can be sideband-preserving or sideband-inverting. This
makes no difference when receiving analog FM (both sound the same) or AX.25
(which uses a differential protocol).

However, some satellites which use FSK (AAUSAT-4 and GOMX-3, for instance) need
a sideband-preserving FM receiver. If your receiver is sideband-inverting, you
can use set `--invert=-1` while running the `.py` file or edit the corresponding
parameter in the `.grc` file to invert the signal again in the decoder and
recover the original signal with the correct sidebands.

## Other hints

To run the decoder and save the output to a file, it is possible to do something
like

```bash
python2 -u aausat_4.py | tee /tmp/aausat4.log
```

This will both print the beacons in real time and also save all the output to
the text file `/tmp/aausat4.log`.
