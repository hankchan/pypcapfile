pypcapfile
==========

pypcapfile is a pure Python library for handling libpcap savefiles. 


Installing
----------

The easiest way to install is from 
[pypi](http://pypi.python.org/pypi/pypcapfile/):

```bash
sudo pip install pypcapfile
```

Note that for pip, the package name is `pypcapfile`; in your code you will need to 
import `pcapfile`.

Alternatively, you can install from source. Clone the repository, and run setup.py with 
an install argument:

```bash
git clone git://github.com/kisom/pypcapfile.git
cd pypcapfile
./setup.py install
```

This does require the Python [distutils](http://docs.python.org/install/) to be
installed.


Introduction
------------

The core functionality is implemented in `pcapfile.savefile`:

```python
>>> from pcapfile import savefile
>>> testcap = open('test.pcap')
>>> capfile = savefile.load_savefile(testcap, verbose=True)
[+] attempting to load test.pcap
[+] found valid header
[+] loaded 11 packets
[+] finished loading savefile.
>>> print capfile
little-endian capture file version 2.4
snapshot length: 65535
linklayer type: LINKTYPE_ETHERNET
number of packets: 11
```

You can take a look at the packets in `capfile.packets`:
```python
>>> pkt = capfile.packets[0]
>>> pkt.raw()
<binary data snipped>
>>> pkt.timestamp
1343676707L
```

Right now there is very basic support for Ethernet frames and IPv4 packet 
parsing. 

Automatically decoding layers
-----------------------------

The `layers` argument to `load_savefile` determines how many layers to 
decode; the default value of 0 does no decoding, 1 will load only the link 
layer, etc... For example, with no decoding:

```python
>>> from pcapfile import savefile
>>> from pcapfile.protocols.linklayer import ethernet
>>> from pcapfile.protocols.network import ip
>>> import binascii
>>> testcap = open('samples/test.pcap')
>>> capfile = savefile.load_savefile(testcap, verbose=True)
[+] attempting to load samples/test.pcap
[+] found valid header
[+] loaded 3 packets
[+] finished loading savefile.
>>> eth_frame = ethernet.Ethernet(capfile.packets[0].raw())
>>> print eth_frame
ethernet from 00:11:22:33:44:55 to ff:ee:dd:cc:bb:aa type IPv4
>>> ip_packet = ip.IP(binascii.unhexlify(eth_frame.payload))
>>> print ip_packet
ipv4 packet from 192.168.2.47 to 173.194.37.82 carrying 44 bytes
```

and this example:

```python
>>> from pcapfile import savefile
>>> testcap = open('samples/test.pcap')
>>> capfile = savefile.load_savefile(testcap, layers=1, verbose=True)
[+] attempting to load samples/test.pcap
[+] found valid header
[+] loaded 3 packets
[+] finished loading savefile.
>>> print capfile.packets[0].packet.src
00:11:22:33:44:55
>>> print capfile.packets[0].packet.payload
<hex string snipped>
```

and lastly:
```python
>>> from pcapfile import savefile
>>> testcap = open('samples/test.pcap')
>>> capfile = savefile.load_savefile(testcap, layers=2, verbose=True)
>>> print capfile.packets[0].packet.payload
ipv4 packet from 192.168.2.47 to 173.194.37.82 carrying 44 bytes
```

The IPv4 module (`ip`) currently only supports basic IP headers, i.e. it 
doesn't yet parse options or add in padding.

The interface is still a bit messy.


Future planned improvements
---------------------------

* IP option handling
* IPv6 support
* TCP and UDP support
* ARP support


TODO
----

0. write unit tests
0. add `__repr__` method that shows all of the values of the fields in IP packets
and Ethernet frames.


See also
--------

* The project's [PyPi page](http://pypi.python.org/pypi/pypcapfile).
* The project's [Sphinx](http://sphinx.pocoo.org/) 
[documentation on PyPI](http://packages.python.org/pypcapfile/)
* The [libpcap homepage](http://www.tcpdump.org)

Contributors
------------
pycapfile was written by [Kyle Isom](https://github.com/kisom/).

[Joshua Chia](https://github.com/jchia/) provided a patch to use the standard
Python file objects instead of a path to the file; this allows transparent
handling of certain types of compressed files.
