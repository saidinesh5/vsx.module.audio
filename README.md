vsx.module.audio
================

An Audio module for VSXu with a fresh start.

It currently uses RtAudio Library for input and hence supports:

1) Windows : DirectSound, ASIO
2) Linux : ALSA, PULSEAUDIO, JACK
3) OSX: CoreAudio, JACK


Install:
--------

Install the needed dependencies (for eg. libpulse-dev on debian based systems with pulseaudio or libasound-dev, and vsxu-dev) and then build the library as follows.

    git clone git://github.com/saidinesh5/vsx.sample.module.git
    cd vsx.sample.module
    mkdir build
    cd build
    cmake ..
    make
    make install

This automatically installs the plugin to the vsxu plugins directory.
You may need to run the "make install" command with administrator privileges.

NOTE: This library is NOT meant to be run with VSXu Player. As the player spawns a new module for every visual, and this module spawns a new thread per instance.