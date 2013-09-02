
**A readonly subversion clone of this project exists at http://code.google.com/p/bully/source/checkout **

**Prebuilt binary(s) can be downloaded from google code at http://code.google.com/p/bully/downloads/list **

# OVERVIEW

Bully is a new implementation of the WPS brute force attack, written in C. It is conceptually identical
to other programs, in that it exploits the (now well known) design flaw in the WPS specification. It has
several advantages over the original reaver code. These include fewer dependencies, improved memory and
cpu performance, correct handling of endianness, and a more robust set of options. It runs on Linux, and
was specifically developed to run on embedded Linux systems (OpenWrt, etc) regardless of architecture.

Bully provides several improvements in the detection and handling of anomalous scenarios. It has been
tested against access points from numerous vendors, and with differing configurations, with much success.


# DEPENDENCIES

Bully requires libpcap and libssl. It uses WPS functionality written by Jouni Malinen; that source code
is included in this repro for simplicity, as are libcrypto and several other sources that provide needed
functionality.

Because Bully stores randomized pins and session data in normal files, there is no need for any database
functionality.


# INSTALLATION

Bully can be built and installed by running:

	~/bully$ cd src
	~/bully/src$ make
	~/bully/src$ sudo make install


# OPENWRT BUILDS

A Makefile tested in Kamikaze r18801 (2.6.26) is provided in the repository root directory. Porting to
Backfire or another OpenWrt variant should be fairly straightforward.

Assuming you have ~/kamikaze as your openwrt directory and ~/bully for bully, the following steps should
get you up and running:

	cd ~/kamikaze
	mkdir package/bully
	cp -rf ~/bully/* ~/kamikaze/package/bully
	make menuconfig

Navigate to Network-->wireless and select bully (module or built-in), exit and save. If you elected to
build as a package, type

	make package/bully/{clean,compile} V=99
	scp bin/packages/<arch>/bully_1.1-1_<arch>.ipk root@<router-ip>/tmp
	ssh root@<router-ip>

enter router password,

	opkg install /tmp/bully*ipk

If you chose to build bully into your firmware, make and install it as you normally would.


# USAGE

Ensure that you are root, and are using wireless hardware that is capable of injection with a monitor mode
interface.

<pre><code>
usage: bully \<options\> interface
Required arguments:
  interface      : Wireless interface in monitor mode (root required)
  -b, --bssid macaddr    : MAC address of the target access point
Or
  -e, --essid string     : Extended SSID for the access point
Optional arguments:
  -c, --channel N[,N...] : Channel number of AP, or list to hop [b/g]
  -l, --lockwait N       : Seconds to wait if the AP locks WPS   [43]
  -p, --pin N            : Index of pin to start at (7 digits) [Auto]
  -s, --source macaddr   : Source (hardware) MAC address      [Probe]
  -v, --verbosity N      : Verbosity level 1-3, 1 is quietest     [3]
  -w, --workdir          : Location of pin/session files  [~/.bully/]
  -5, --5ghz             : Hop on 5GHz a/n default channel list  [No]
  -F, --fixed            : Fixed channel operation (do not hop)  [No]
  -S, --sequential       : Sequential pins (do not randomize)    [No]
  -T, --test             : Test mode (do not inject any packets) [No]
Advanced arguments:
  -a, --acktime N        : Acknowledgement and pcap timeout (ms) [25]
  -r, --retries N        : Resend packets N times when not acked  [2]
  -m, --m13time N        : M1/M3/Initial beacon timeout (ms)   [2000]
  -t, --timeout N        : Timeout for Auth/Assoc/Id/M5/M7 (ms) [200]
  -1, --pin1delay M[,N]  : Delay M seconds every Nth nack at M5 [0,1]
  -2, --pin2delay M[,N]  : Delay M seconds every Nth nack at M7 [5,1]
  -A, --noacks           : Disable ACK check for sent packets    [No]
  -C, --nocheck          : Skip CRC/FCS validation (performance) [No]
  -D, --detectlock       : Detect WPS lockouts unreported by AP  [No]
  -E, --eapfail          : EAP Failure terminate every exchange  [No]
  -L, --lockignore       : Ignore WPS locks reported by the AP   [No]
  -M, --m57nack          : M5/M7 timeouts treated as WSC_NACK's  [No]
  -N, --nofcs            : Packets don't contain the FCS field [Auto]
  -P, --probe            : Use probe request for nonbeaconing AP [No]
  -R, --radiotap         : Assume radiotap headers are present [Auto]
  -W, --windows7         : Masquerade as a Windows 7 registrar   [No]
  -h, --help             : Display this help information
</pre></code>


# DESCRIPTION OF ARGUMENTS

      -c, --channel N[,N...]

		Channel number, or comma separated list of channels to hop on. Some AP's will switch
		channels periodically. This option allows bully to reacquire an AP and continue an attack
		without intervention. Note that using channel hopping will typically slow an attack,
		especially when the AP's signal is weak, because time is spent scanning channels instead
		of testing pins. If no channel is provided, bully will hop on all channels.

      -l, --lockwait N

		Number of seconds to wait when an AP locks WPS. Most AP's will lock out for 5 minutes, so
		the default value is 43 seconds. This will cause bully to sleep 7 times during a lockout
		period for a total of 301 seconds.

      -p, --pin N

		This is the starting pin number (sequential runs) or the index of the starting pin number
		(randomized runs).  This is typically handled for you automatically, i.e. an interrupted
		session will resume after the last pin that was successfully tested.

      -s, --source macaddr

		The source MAC address to embed in packets sent to the AP. Not all wireless cards can be
		used to spoof the source MAC address like this, but the option is provided for chipsets
		that allow it. When not provided, the wireless interface is probed to retrieve the MAC.

      -v, --verbosity N

		Verbosity level. 1 is the quietest, displaying only unrecoverable error information. Level
		3 displays the most information, and is best used to determine exactly what is happening
		during a session.

      -w, --workdir

		Working directory, where randomized pins and session files are stored. Session files are
		created in this directory based on the BSSID of the access point. Only one set of randomized
		pins is created, and is used for all sessions. If you want to regenerate the pin file, simply
		delete it from this directory; however incomplete runs that used the deleted file will not
		be restartable. The default directory is ~/.bully/

      -5, --5ghz

		Use 5 GHz (a/n) channels instead of 2.54 GHz (b/g) channels. Untested.

      -F, --fixed

		Uhm, deprecated. The right way to do this is specify a single channel using --channel. Idk.

      -S, --sequential

		By default, pins are randomized. This options allows pins to be tested sequentially.

      -T, --test

		Test mode. No packets are injected. Can be used to validate arguments, determine if an
		access point is visible and has WPS enabled, generate a randomized pin file, or create a
		session file for the access point.

      -a, --acktime N

		This timeout period is used when waiting for acknowledgements for sent packets, and is also
		used as the pcap timeout value. Default is 25 ms. Increase this value on high traffic channels,
		or if you see numerous "sent packet not acknowledged" messages.

      -r, --retries N

		How many times do we resend packets when they aren't acknowledged? Default is 3. The idea is to
		make a best effort to ensure the AP receives every packet we send, rather than have transactions
		fail and restart due to a missed packet.

      -m, --m13time N

		Timeout period for the first beacon from the AP, and WPS messages M1 and M3. Default is 2000 ms.
		This is a relatively long period, on the order of 100x longer than an acknowledgement, because
		these messages involve extensive computation on the AP. Increase this value for slower (or
		busier) access points.

      -t, --timeout N

		Standard timeout period for all remaining packets types (authentication, association, request
		identity, M5 and M7).  Default is 200 ms. Increase on busy channels/AP's.

      -1, --pin1delay M[,N]

		Delay M seconds for every Nth NACK at M5. The default is 0,1 (no delay). Some access points
		get overwhelmed by too many successive WPS transactions, and can even crash if we don't dial
		things back a bit. This is the delay period to use during the first half of the pin.

      -2, --pin2delay M[,N]

		Delay M seconds for every Nth NACK at M7. The default is 0,1 (no delay). Some access points
		handle transactions through M4 easily, only to fall down on too many successive M6 messages.
		This is the delay period to use during the second half of the pin.
		
      -A, --noacks

		Turn off acknowledgement processing for all sent packets. Useful if you are sure the AP is
		receiving packets even though bully can't see acknowledgements. You might need this for a USB
		wifi adapter that processes acknowledgements and drops them before libpcap ever sees them.

      -C, --nocheck

		Turn off frame check sequence processing. We can improve performance somewhat by making the
		dubious assumption that all packets we receive are valid. See also --nofcs below.

      -D, --detectlock

		Certain access points do not indicate that they have locked WPS in their beacon IE tags, but
		summarily ignore all WPS transactions for a period of time. With this option, we can detect the
		condition and sleep for --lockdelay seconds before resuming. In the interests of remaining
		undetected, there is no point in broadcasting 5 minutes worth of unanswered EAP START messages.

      -E, --eapfail

		Send EAP FAIL messages after each transaction. Some AP's get confused when they don't see this.

      -L, --lockignore

		Ignore WPS lock conditions reported in beacon information elements (don't sleep).

      -M, --m57nack

		Treat M5 and M7 timeouts as NACK's, for those access points that don't send them but instead
		drop the transaction. When using this option you will probably want to increase the --timeout 
		value, so that bully doesn't incorrectly assume a pin is incorrect due to a delayed message.

      -N, --nofcs

		Some wireless hardware will have done the work of checking and stripping the FCS from packets
		already. Bully usually detects this and adjusts accordingly, but the option is here if you need
		to force it.

      -P, --probe

		Bully uses beacons to examine the WPS state of an access point. For nonbeaconing AP's, send
		directed probe requests and use the resulting probe responses instead. Requires --essid.

      -R, --radiotap

		Assume radiotap headers are present in received packets. This is useful in cases where presence
		of radiotap headers is incorrectly reported or detected.

      -W, --windows7

		Masquerade as a Windows 7 registrar.

      -h, --help

		Display onscreen help.

