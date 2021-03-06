# Network

## Requirements

This plugin has no requirements or dependencies.
Please be aware that ports need to be higher than 1025 to work. Otherwise an access error might occur. 
Ports up to 1024 are _well defined ports_ and are normally reserved to the system usage.

## Configuration

### plugin.conf (deprecated) / plugin.yaml

```
[nw]
    class_name = Network
    class_path = plugins.network
    # ip = 0.0.0.0
    # port = 2727
    tcp = yes
    tcp_acl= 127.0.0.1 | 192.168.0.34
    # udp = no
    # udp_acl= *
```

```yaml
nw:
    class_name: Network
    class_path: plugins.network
    # ip: 0.0.0.0
    # port: 2727
    tcp: 'yes'
    tcp_acl:
      - 127.0.0.1
      - 192.168.0.34
    # udp: 'no'
    # udp_acl: '*'
```

### Attributes

  * `ip`: specifies the listening IP address. By default it listens on all addresses.
  * `port`: specifies the listening port for generic incoming TCP and UDP connections. By default it listens on 2727.
  * `tcp`: by default the plugin doesn't accept incoming TCP connections. You have to set this attribute to 'yes' to accept them.
  * `tcp_acl`: with this attribute you could specify a list or a single IP address to allow TCP updates from. By default it accepts every incoming request.
  * `udp`: by default the plugin doesn't accept incoming UDP connections. You have to set this attribute to 'yes' to accept them.
  * `udp_acl`: with this attribute you could specify a list or a single IP address to allow UDP updates from. By default it accepts every incoming request.
  * `http`: port to listen for HTTP GET request
  * `http_acl`: with this attribute you could specify a list or a single IP address to allow HTTP updates from. By default it accepts every incoming request.


### items.conf (deprecated) / items.yaml

#### nw
If this attribute is set to 'yes' you could update this item with the generic listener (TCP and/or UDP).

```
[test]
    [[item1]]
        type = str
        nw = yes
```

```yaml
test:

    item1:
        type: str
        nw: 'yes'
```

#### nw_acl

Like the generic tcp_acl/udp_acl a list or single IP address to limit updates from.
This attribute is valid for TCP and UDP and overrides the generic tcp_acl/udp_acl.

#### nw_udp_listen/nw_tcp_listen

You could specify the `nw_udp_listen` and `nw_tcp_listen` attribute to an item to create a dedicated listener. The argument could be a port or ip:port.

```
[test]
    [[item1]]
        type = str
        # bind to 0.0.0.0:7777 (every IP address)
        nw_tcp_listen = 7777

    [[item2]]
        type = str
        # bind to 0.0.0.0:7777 and 127.0.0.1:8888
        nw_udp_listen = 127.0.0.1:8888
```

```yaml
test:

    item1:
        type: str
        # bind to 0.0.0.0:7777 (every IP address)
        nw_tcp_listen: 7777

    item2:
        type: str
        # bind to 0.0.0.0:7777 and 127.0.0.1:8888
        nw_udp_listen: 127.0.0.1:8888
```

If you send a TCP/UDP packet to the port, the corresponding item will be set to the TCP/UDP payload.
``$ echo teststring | nc -u 127.0.0.1 8888`` would set the value of ``item2`` to ``teststring``.

#### nw_udp_send
This attribute allows you to specify a host and port to send item updates to.

```
[test]
    [[item1]]
        type = str
        nw_udp_send = 11.11.11.11:7777  # sends an UDP packet with the item value as payload

    [[item2]]
        type = str
        nw_udp_send = 11.11.11.11:7777=special data  # sends an UDP packet with 'special data' as payload

    [[item3]]
        type = str
        nw_udp_send = 11.11.11.11:7777=command: itemvalue  # sends an UDP packet with 'command: ' and the current item value as payload
```

```yaml
test:

    item1:
        type: str
        nw_udp_send: 11.11.11.11:7777    # sends an UDP packet with the item value as payload

    item2:
        type: str
        nw_udp_send: "11.11.11.11:7777=special data    ## sends an UDP packet with 'special data' as payload"

    item3:
        type: str
        nw_udp_send: "11.11.11.11:7777=command: itemvalue    ## sends an UDP packet with 'command: ' and the current item value as payload"
```

### logic.conf

You could use the same network attributes as in items.conf to trigger logics.

In the context of the KNX plugin the trigger dictionary consists of the following elements:

* trigger['by']     protocol (tcp, udp, http)
* trigger['source']     IP adress of the sender
* trigger['value']     payload 


## Usage

The generic listener accepts a simple message format: `key|id|value`.
Currently are three different keys supported:

  * `item|item.path|value`
  * `logic|logic_name|value`
  * `log|loglevel|message` # loglevel coud be info, warning or error

```bash
# send a udp message to set the item 'network.incoming' to '123'
$ echo "item|network.incoming|123" | nc -uw 1 XX.XX.XX.XX 2727`

# send a tcp message to trigger the logic 'say' with 'hello'
$ echo "logic|say|hello" | nc -w 1 XX.XX.XX.XX 2727`

# send a udp message to add an log entry with loglevel 'warning' and the message 'lost internet connection'
$ echo "log|warning|lost internet connection" | nc -uw 1 XX.XX.XX.XX 2727`

# http request to set the item 'network.incoming' to '123'
$ wget "http://XX.XX.XX.XX:8090/item|network.incoming|123"
```

## Functions

### udp(host, port, data)

``sh.nw.udp('192.168.0.5', 9999, 'turn it on')`` would send ``turn it on`` to ``192.168.0.5`` port ``9999``. Simple, isn't it?
