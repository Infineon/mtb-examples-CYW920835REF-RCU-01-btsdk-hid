-------------------------------------------------------------------------------
BLE Remote Control
-------------------------------------------------------------------------------

Overview
--------

The BLE RCU reference remote application is a single-chip SoC compliant with HID over GATT Profile
(HOGP). Supported features include key, and microphone (voice over HOGP).

During initialization, the app registers with LE stack, WICED HID Device Library and
keyscan and external HW peripherals to receive various notifications including
bonding complete, connection status change, peer GATT request/commands and
interrupts for key pressed/released.
After successful pairing, press or release any key, a key report will be sent to the host.
On connection up or battery level changed, a battery report will be sent to the host.
When the battery level is below shutdown voltage, the device will critical shutdown.
When the user presses and holds the audio key, voice streaming starts until the user releases
the microphone key.

Features demonstrated
---------------------
 - GATT database and Device configuration initialization
 - Registration with LE stack for various events
 - Sending HID reports to the host
 - Processing write requests from the host
 - Low power management
 - Over the air firmware update (OTAFWU)

 Instructions
-------------
To demonstrate the app, walk through the following steps -

1. Plug the 20835 Remote Control HW into your computer.

2. Build and download the application to the Remote Control HW. In case if the device cannot be
   detected for programming after several trials, it could mean the firmware is corrupted and
   the device is not functional. In this case, we need a way to recover the device. Place a push
   button between GND and SPI_MOSI pin at the PUART connector. To recover the device, unplug the
   device first. While pressing the button (shorting GND and SPI_MODI pin), powering up the device by
   step 1 to plug the device. Then, the device should be in programming mode and ready for step 2.

3. Unplug and plug to reset the device. Right after reset, the red LED should do fast blink
   5 times to indicate the device is reset and firmware is running.

4. Put device to pairing mode by press HOME key and hold for 10 seconds.
   The red LED should be turned on while holding HOME key and counting. After 10 seconds
   expires, The red LED should be turned off and the green LED will start slow blink to indicate
   the device is in pairing mode. This procedure will also erase prior paired host information.

5. Pair the remote to a host (Windows, Android, or iOS).

6. Once connected, it becomes the remote control of the host. The basic keys, such as navigation
   keys should be supported by most of the OSs; however, some keys, such as Menu, Home key function
   may not be supported by the OS. The audio function is very host dependent. The encryption method
   also needs to match. For example, to work with Android TV, ENABLE_ANDROID_AUDIO flag must be turned
   on. It will use ADPCM encoding. For this remote, we use analog MIC, therefore, ENABLE_DIGITAL_MIC
   should be turned off.

Using ClientControl tool:
NOTE: Make sure you the compile flag "TESTING_USING_HCI=1" is turned on. This allows PC application,
ClientCiontrol, to control the device via HCI_UART port.

1. Run ClientControl.exe.
   In ModusToolbox, select CYW920835REFRCU device first. At "Tools" section, double click on "ClientControl"
   to launch the application.

2. Choose 3M as Baudrate and select the serial port in the ClientControl tool window.

3. Establish communication between ClientControl and the device.

   Power cycle the remote (unplug and plug the device) and within 2 seconds, open the port.
   When the device is communicating with ClientControl, the "HIDD device" page tabs should become available.
   In case if the open was not opened within 2 seconds, the communication may not come up. In this case,
   close the port and do step 3 again.
   An alternate way to bring up the communication is to install a reset button between TP1
   and Ground(TP22). Close the port first and press the reset button instead of power cycling the device.
   If the port is not closed, the reset may cause a core dump; however, as long as the HID tab can be enabled, proceed to step 4.

4. Click "Enter Pairing Mode" or "Connect" to start LE advertising, then pair with a host.

5. Once connected, it becomes the remote control of the TV.
   Use left, right, up, down, ESC, Enter keys to navigate the host.
   Use the audio button (click and hold) to stream audio.
   Since this remote does not support motion and IR. Those buttons take no action.

Application Settings
--------------------
TESTING_USING_HCI
    Use this option for testing with Bluetooth Profile Client Control. The Client
    Control UI can be used to provide input. When this option is enabled, the
    device will not enter SDS/ePDS for power saving.

OTA_FW_UPGRADE
    Use this option for enabling firmware upgrade over the air (OTA) capability.

  OTA_SEC_FW_UPGRADE
    Use this option for a secure OTA firmware upgrade. OTA_FW_UPGRADE option must be
    enabled for this option to take effect.

AUTO_RECONNECT
    Use this option to enable auto-reconnect. When enabled, the device will automatically
    try to reconnect when the link is disconnected. When this option is disabled, the device
    will enter sleep when the link is disconnected to conserve power.

DISCONNECTED_ENDLESS_ADV
    Use this option to enable disconnected endless advertisements. When the device is
    in an advertisement state, the advertisement will not expire when this option is
    turned on. Otherwise, the advertisement expires in 60 sec to conserve power.

    When both AUTO_RECONNECT and DISCONNECTED_ENDLESS_ADV are turn on, once the device
    is paired, it will always attempt to reconnect to the host forever. Thus, when host
    is not available, the battery can drain quickly.

SKIP_PARAM_UPDATE
    Use this option to skip to send link parameter update request.
    When this option is disabled, if the peer device (master) assigned link parameter
    is not within the device's preferred range, the device will send a request for
    the desired link parameter change. This option can be enabled to stop the device
    from sending the request and accept the given link parameter as is.

    Background:
    In some OS (peer host), after the link is up, it continuously sends different
    parameter of LINK_PARAM_CHANGE over and over for some time. When the parameter
    is not in our device preferred range, the firmware was rejecting and renegotiating
    for a new preferred parameter. It can lead up to endless and unnecessary overhead
    in link parameter change. Instead of keep rejecting the link parameter, by using
    this option, we accept the peer requested link parameter as it and starts a timer to
    send the final link parameter change request later when the peer host settles down
    in link parameter change.

START_ADV_ON_POWERUP
    Use this option to start the advertisement after power-up (cold boot). By enabling
    this option, after power-up, the device will automatically try to connect to the
    paired host if it is already paired. If it is not paired, it will enter discovery
    mode for pairing.

ENABLE_CONNECTED_ADV
    Use this option to allow advertisement for new host pairing while
    connected to a host.

ASSYMETRIC_SLAVE_LATENCY
    Use this option to enable asymmetric slave latency.

    Background:
    In the early days, some HID host devices will always reject HID slave's link
    parameter update request. Because of this, the HID device will end up consuming
    high power when slave latency was short. To work around this issue, we use
    Asymmetric Slave Latency method to save power by waking up only at multiple
    time of the communication anchor point. When this option is enabled,

    1.  We do not send LL_CONNECTION_PARAM_REQ.
    2.  We simply start Asymmetric Slave Latency by waking up at multiple times
        of given slave latency.

    Since this is not a standard protocol, we do not recommend enabling this
    option unless if it is necessary to save power to work around some HID hosts.

ENABLE_AUDIO
  Use this option to enable audio function to send voice over HOGP (HID over
  GATT Protocol). By default, mSBC encoding method is used unless one of the
  following option is enabled:

  OPUS_CELT_ENCODER: use Opus Celt encoder
  ADPCM_ENCODER: use ADPCM encoder

    If both OPUS_CELT_ENCODER and ADPCM_ENCODER are defined, Opus Celt encoding
    method is used.

  ENABLE_DIGITAL_MIC
    Use this option to disable the digital microphone (ENABLE_DIGITAL_MIC=0).
    ENABLE_AUDIO option must be enabled for this option to take effect.

ENABLE_ANDROID_AUDIO
  When this option is enabled, the Android TV Voice Service is enabled. It forces
  the following options:

  ENABLE_AUDIO: Audio is enabled
  ADPCM_ENCODER: Forced to use ADPCM encoder (forced OPUS_CELT_ENCODER=0)

Note on behavior with Win10:
1. Voice -- acting as 'Search'
   On pressing and releasing the 'Audio' key, the 'Find' window opens.
   Again pressing and releasing the 'Audio' button, Windows search enabled.
2. When a document is open, pressing and releasing 'right', 'left', 'up', 'down'
   and 'Enter' keys work.
3. On pressing and releasing the 'Home' key, the default browser opens.
4. On pressing and releasing 'Power', 'Back' and, 'Menu' keys, there is no response
   at host side.
5. When audio is streaming for music stored locally, only 'Play' and 'Pause'
   key works. 'Fast Forward', 'Rewind', 'Vol+', 'Vol-' and 'Mute' keys do not work.
6. When streaming from YouTube, 'Right' and 'Left' keys work as 'Fast Forward'
   and 'Rewind' respectively. 'Up' and 'Down' keys work as Volume Up and Volume Down.
   'Fast Forward', 'Play/Stop' 'Rewind' brings up Media pannel, but only 'Play/Stop' will work.

Note on behavior with iPhone:
1. Voice -- acting as 'Search'
2. Navigations keys, Enter key should work in a text editor (such as email editing)
3. Back -- acting as 'Cancel' (For example, in the email editor, it will cancel editing)
4. Home, Menu -- work as Home button
3. Rewind, play/stop, and fast forward keys work on Youtube while playing a video.
4. Power, Vol+, Vol-, Mute keys have no action.
