# LIC: GPL

$Id$

This package lets you connect a Linux machine to an ISP that uses PPPoE.
PPPoE is used by many DSL providers and some wireless providers.

Follow these steps and you should have your PPPoE service up and running.

0. Install the rp-pppoe-software
--------------------------------

You should have already done this by the time you're reading this.  If not,
go back and read README.

1. Set up your Ethernet hardware
--------------------------------

First, make sure the Ethernet card you intend to use with the modem is
visible to the Linux kernel.  Just how to do this is beyond the scope
of this document.  However, if the card is the only Ethernet card in
the system, executing:

    ```sh
	$ ifconfig en0
	```

should display something like this:
    ```sh
	en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
            options=400<CHANNEL_IO>
            ether f8:ff:c2:37:07:82
            inet6 fe80::1055:ab4e:b945:806c%en0 prefixlen 64 secured scopeid 0x6
            inet 192.168.31.11 netmask 0xffffff00 broadcast 192.168.31.255
            nd6 options=201<PERFORMNUD,DAD>
            media: autoselect
            status: active
    ```

plust some more lines.  Your HWaddr will be different.  As long as you see
the HWaddr line, your card should be working.

DO NOT assign an IP address to the Ethernet card.  DO NOT configure the
card to come up at boot time.

2. Configure various files
--------------------------

Several files need editing.  The easiest way to do this is to run
the following command as root:

	pppoe-setup

Answer the questions and you should be all set.  If you want to know what
goes on behind the scenes, continue reading this document.  If you don't
care and your connection works, stop reading. :-)

3. Edit pap-secrets
-------------------

Edit the "pap-secrets" file, inserting your proper user-ID and password.
Install the file (or copy the relevant lines) to /etc/ppp/pap-secrets.
Your ISP may use CHAP authentication.  In this case, add the line to
/etc/ppp/chap-secrets.

4. Edit /etc/ppp/pppoe.conf
-----------------------------

The file /etc/ppp/pppoe.conf contains configuration information for the
DSL connection.  You need to edit the following items:

- Change ETH=eth1 to the correct Ethernet device for your modem.
- Change USER=bxxxnxnx@sympatico.ca to your proper DSL user-ID.

Don't edit any of the other settings unless you're an expert.

5. Set up DNS
-------------

If you are using DNS servers supplied by your ISP, edit the file
/etc/resolv.conf to contain these lines:

    ```sh
	nameserver ip_addr_of_first_dns_server
	nameserver ip_addr_of_second_dns_server
	```

For example:

    ```sh
	nameserver 204.101.251.1
	nameserver 204.101.251.2
	```


6. Firewall your machine
------------------------

MAKE SURE YOU FIREWALL YOUR MACHINE.  A sample firewall script is given
in the shell script "firewall"  To install the script:

a) Copy it to /etc/rc.d/init.d/firewall
b) Type: chkconfig firewall on
c) Start the firewall: sh /etc/rc.d/init.d/firewall start

(The above procedure works ONLY on Red Hat-like systems.)

You may want to tweak the script somewhat.

7. Commands to control the PPPoE link
-------------------------------------

As root, bring up the link by typing:   pppoe-start
As root, bring down the link by typing: pppoe-stop

That's it!

--
Dmitry Mamontov <d,slonyara@gmail.com>
Dianne Skoll <dfs@roaringpenguin.com>

PROBLEMS!  DIANNE, IT DOESN'T WORK!
---------------------------------

Here are some problems PPPoE users have encountered.

-----------------------------------------------------------------------------
A) Can't see the Ethernet interface

Well, I can't really help you here.  To use these instructions, you must
have Linux working to the point where it recognizes your Ethernet card.
If you type "ifconfig enx" and you get back a HWAddr value, your Ethernet
card is probably OK.  But I really can't help with hardware configuration
issues.

-----------------------------------------------------------------------------
B) Connection seems to come up, but I can't browse the web or ping anything

You probably don't have DNS set up.  See step 6.

-----------------------------------------------------------------------------
C) Can't compile PPPoE

Make sure you have "make", the C compiler and all development header
files installed.  I only test rp-pppoe on MacOS.  It might not work on
*BSD and probably won't work on any other version of UNIX.

-----------------------------------------------------------------------------
D) pppd complains about (i) "unknown option pty" or (ii) "pty option precludes
   specifying device name"

(i) Your pppd is too old.  You need at least 2.4.2.
(ii) Your /etc/ppp/options file is not empty.  Empty it!

-----------------------------------------------------------------------------
E) pppoe dies with the log message "Message too long"

You set the MTU of the Ethernet interface connected to the DSL modem
to less than 1500.  Don't do that.

-----------------------------------------------------------------------------
F) Internal hosts can't see the Internet

Do you have masquerading set up?  I can't help you in great detail, but
see the IPCHAINS-HOWTO and the IP-Masquerade mini-HOWTO.

-----------------------------------------------------------------------------
G) Authentication fails

Make sure you have the right secret in /etc/ppp/pap-secrets.  Your ISP
may be using CHAP; it won't hurt to copy the line to /etc/ppp/chap-secrets.

Also, MAKE SURE that /etc/ppp/options is EMPTY.  The "pppoe-connect" script
supplies all required options on the command line; additional options
in /etc/ppp/options may mess things up.

-----------------------------------------------------------------------------
H) VPN software does not work

If you are using VPN software on a Windows or Linux machine with another
Linux machine running PPPoE as the gateway, you MUST NOT use the "-m" option
to pppoe.  This alters IP packets, which will break any VPN which uses IPSec.
In /etc/ppp/pppoe.conf, set CLAMPMSS to "no".  You'll also have to reduce
the MTU on the hosts behind the gateway to 1452.

-----------------------------------------------------------------------------
I) I can browse some web sites just fine, but others stall forever.

There is probably a buggy router or firewall between you and the Web server.
One possible workaround:  In /etc/ppp/pppoe.conf, find the line which reads:

	CLAMPMSS=1412

Try lowering the 1412 until it works (go down in steps of 100 or so.)  Each
time you lower the value, you have to restart your connection like this:

	pppoe-stop; pppoe-start

This should work around buggy routers which do not support Path MTU discovery.

-----------------------------------------------------------------------------
J) Whenever I connect using DSL, my internal LAN no longer sees the gateway

You are more than likely running a 2.0.X Linux kernel.  To solve this
problem, give the Ethernet card connected to the DSL modem a fake IP
address.  For example, if en0 is your internal LAN card and eth1 goes to
the DSL modem, do something like this:

    ```sh
	ifconfig en1 10.0.0.1 netmask 255.255.255.0
	```

(You may have to choose a different IP address; experiment.)
-----------------------------------------------------------------------------
K) How can I run a script every time I connect and get a new IP address?

Put the script in /etc/ppp/ip-up.  See the pppd(8) man page.
-----------------------------------------------------------------------------
L) Nothing works!

You may need to put your Ethernet card in half-duplex, 10Mb/s mode to
work with the DSL modem.  You may have to run a DOS program to do this,
or pass special parameters to the Linux driver.

Some providers object to attempts to set the MRU or MTU.  Try removing
"mtu 1492 mru 1492" from PPP_STD_OPTIONS in the pppoe-connect script.
This problem has been seen with an ISP in Hong Kong.

Your DSL provider may be using non-standard PPPoE frames or require
something special in the Service-Name field.  If you have two computers,
you can try sniffing out these values with the "pppoe-sniff" program.
Type "man pppoe-sniff" for details.  If you don't have two computers,
you'll have to ask your DSL provider if it uses non-standard PPPoE frames
or special Service-Name fields.  Good luck getting an answer...

If pppoe-sniff indicates that nothing is amiss, make sure the Ethernet
card associated with the DSL modem does NOT have a valid IP address.
(NOTE: For 2.0 kernels, you may have to give it a fake IP address
which is not on your internal subnet.  Something like 192.168.42.42
might work if you are not using 192.168.42.*)

If you are using synchronous PPP on a slow machine, try switching to
asynchronous PPP.

Make sure no entries in the routing table go through the Ethernet card
connected to the DSL modem.  You might want to add these lines in
pppoe-connect:

    ```sh
	ifconfig enx down
	ifconfig enx up mtu 1500
	```

which should reset things to sane values.
