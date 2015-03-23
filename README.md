# Reverse Engineered UniFi Protocol

This document describes the reverse-engineered protocol Ubiquity Unifi APs use to communicate with their controller.

It's based on what I've observed; don't rely on it.


## Protocol


### Discover

A factory-reset AP will send discovery packets:

- as broadcast to 255.255.255.255
- as multicast to 233.89.188.1

The packets consist of a fixed header and some fields in Type-Length-Value (TLV) format.
All numbers are in network byte order (big endian).

Header:

- 2 Byte: todo
- 2 Byte: Payload length

Fields:

- 2 Byte: Type
- 2 Byte: Length
- n Byte: Value

Available fields:

- todo


### Adopt

Via SSH. todo.


### Inform

Every 10s, the AP sends an HTTP POST containing it's current status to the "inform URL".
The controller will either respond with a no-op message or with a command message.
Upon receiving a command message, an AP will execute a command and then send another inform immediately.

For Layer 3 adoption purposes, that's `http://unifi:8080/inform` by default.
It can be overridden by DHCP option 43 (Vendor Specific Information) code 1 (type IP-Address).

Both the requests (AP to controller) and the responses (controller to AP) share the same format.
All numbers are in network byte order (big endian).

- 4 byte: magic bytes `TNBU`
- 4 byte (int): packet version. Assuming 1.
- 6 byte: APs MAC address (even in response sent by controller)
- 2 byte (short): flags
	- bit 0x1: payload is encrypted
	- bit 0x2: payload is compressed
- 16 byte: initialization vector
- 4 byte (int): payload version
- 4 byte (int): payload length in byte
- remaining bytes: payload, maybe compressed, maybe encrypted

Compression is done before encryption (as should be obvious).
By default, the controller will not accept unencrypted packets.

The compression algorithm is ZLIB.
The encryption algorithm is AES in CBC mode with PKCS#7-style padding.

Version 1 payloads are JSON documents.

A request might look like this.

A Unifi UAPs request might look a follows.
Field values have been changed; but their capitalization and lengths haven't.
The controller is `192.168.1.1`, the AP is `192.168.1.2`.

	{u'bootrom_version': u'unifi-v1.5.2.206-g44e4c8bc',
	 u'cfgversion': u'0123456789abcdef',
	 u'connect_request_ip': u'192.168.1.2',
	 u'connect_request_port': u'56288',
	 u'country_code': 0,
	 u'default': False,
	 u'guest_token': u'0123456789ABCDEF01234567890ABCDE',
	 u'has_eth1': False,
	 u'has_poe_passthrough': False,
	 u'hostname': u'UBNT',
	 u'if_table': [{u'full_duplex': True,
	   u'ip': u'0.0.0.0',
	   u'mac': u'01:23:45:67:89:ab',
	   u'name': u'eth0',
	   u'num_port': 1,
	   u'rx_bytes': 1234567,
	   u'rx_dropped': 0,
	   u'rx_errors': 0,
	   u'rx_multicast': 1234,
	   u'rx_packets': 12345,
	   u'speed': 100,
	   u'tx_bytes': 1234567,
	   u'tx_dropped': 0,
	   u'tx_errors': 0,
	   u'tx_packets': 12345,
	   u'up': True}],
	 u'inform_url': u'http://192.168.1.1:8080/inform',
	 u'ip': u'192.168.1.2',
	 u'isolated': False,
	 u'locating': False,
	 u'mac': u'01:23:45:67:89:ab',
	 u'model': u'BZ2',
	 u'model_display': u'UAP',
	 u'radio_table': [{u'athstats': {u'ast_ath_reset': 0,
		u'ast_be_xmit': 101550,
		u'ast_cst': 53,
		u'ast_deadqueue_reset': 0,
		u'ast_fullqueue_stop': 0,
		u'ast_txto': 3,
		u'n_rx_aggr': 0,
		u'n_rx_pkts': 512003,
		u'n_tx_bawadv': 0,
		u'n_tx_bawretries': 0,
		u'n_tx_pkts': 13036,
		u'n_tx_queue': 550,
		u'n_tx_retries': 0,
		u'n_tx_xretries': 0,
		u'n_txaggr_compgood': 0,
		u'n_txaggr_compretries': 0,
		u'n_txaggr_compxretry': 0,
		u'n_txaggr_prepends': 0,
		u'name': u'wifi0'},
	   u'builtin_ant_gain': 0,
	   u'builtin_antenna': True,
	   u'max_txpower': 23,
	   u'min_txpower': 5,
	   u'name': u'wifi0',
	   u'radio': u'ng',
	   u'scan_table': []}],
	 u'required_version': u'2.4.4',
	 u'serial': u'0123456789AB',
	 u'state': 2,
	 u'time': 1424915907,
	 u'uplink': u'eth0',
	 u'uptime': 10442,
	 u'vap_table': [{u'bssid': u'00:00:00:00:00:00',
	   u'ccq': 4772488,
	   u'channel': 1,
	   u'essid': u'vport-0123456789AB',
	   u'id': u'user',
	   u'name': u'ath0',
	   u'num_sta': 0,
	   u'radio': u'ng',
	   u'rx_bytes': 0,
	   u'rx_crypts': 0,
	   u'rx_dropped': 0,
	   u'rx_errors': 0,
	   u'rx_frags': 0,
	   u'rx_nwids': 0,
	   u'rx_packets': 0,
	   u'sta_table': [],
	   u'state': u'INIT',
	   u'tx_bytes': 0,
	   u'tx_dropped': 0,
	   u'tx_errors': 0,
	   u'tx_packets': 0,
	   u'tx_power': 23,
	   u'tx_retries': 0,
	   u'up': False,
	   u'usage': u'uplink'},
	  {u'bssid': u'ba:09:87:65:43:21',
	   u'ccq': 4772480,
	   u'channel': 1,
	   u'essid': u'test',
	   u'id': u'0123456789abcdef01234567',
	   u'name': u'ath1',
	   u'num_sta': 0,
	   u'radio': u'ng',
	   u'rx_bytes': 0,
	   u'rx_crypts': 0,
	   u'rx_dropped': 0,
	   u'rx_errors': 0,
	   u'rx_frags': 0,
	   u'rx_nwids': 4097,
	   u'rx_packets': 0,
	   u'sta_table': [],
	   u'state': u'RUN',
	   u'tx_bytes': 301713,
	   u'tx_dropped': 29169,
	   u'tx_errors': 0,
	   u'tx_packets': 1051,
	   u'tx_power': 23,
	   u'tx_retries': 0,
	   u'up': True,
	   u'usage': u'user'}],
	 u'version': u'3.2.10.2886'}


## Dependencies

My decoder has a few Python libraries as dependencies:

- Padding
- PyCrypto
- pymongo


## How to capture

### With mitmproxy

You can use mitmproxy decode and/or tamper with the packets.

	mitmdump --port 18080 --reverse http://localhost:8080/ -s mitmproxy-extension.py

You can use iptables to redirect incoming packets to mitmproxy.

	-m addrtype -p tcp --dport 8080 --dst-type LOCAL ! --src-type LOCAL -j REDIRECT --to-ports 18080

If you're using firewalld, you can configure this as a direct rule:

	firewall-cmd [--permanent] --direct --add-rule ipv4 nat PREROUTING ...

You shouldn't, but you could also add it to `/etc/firewalld/direct.xml` (before starting firewalld):

	<rule ipv="ipv4" table="nat" chain="PREROUTING" priority="1">...</rule>

You can't see the adoption process yet, since the key for the AP will only
be added to the database *after* we've forwarded the packet.
It'd be possible to fix this by delaying decoding of the packet if no key is available.
One would have to take care to prevent displaying packets in the wrong order.

