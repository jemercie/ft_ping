# ft_ping

Mike Muuss wrote this program in December 1983. Its **name** comes from the **sound** emitted by a **sonar**, since their action is similar (emission of a signal which bounces off a target to return to the sender). Subsequently, David L. Mills provided a retroacronym: “Packet InterNet Groper”. *`packet network groper?🤨`*

Ping uses the ICMP protocol's mandatory **ECHO_REQUEST** datagram to elicit an **ICMP ECHO_RESPONSE** from a *host or gateway*. ECHO_REQUEST datagrams (pings) have an **IP and ICMP header**, followed by a **“struct timeval”** and then an arbitrary number of padding bytes used to fill out the packet.

**ICMP** protocol is **layer 3** bcs it's not used for data transmission or by end users but for troubleshooting and network management.

[useful article](https://gursimarsm.medium.com/customizing-icmp-payload-in-ping-command-7c4486f4a1be)
[man ping inetutils](https://manpages.debian.org/stretch/inetutils-ping/ping.1.en.html)
[protocole ICMP](https://ram-0000.developpez.com/tutoriels/reseau/ICMP/)
## IP header
```c
                1               2
 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8  // bytes
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Flags|     Fragment Offset     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|        Header Checksum        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |
+         Source Address        +
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               |
+      Destination Address      +
|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Options            |
+               +-+-+-+-+-+-+-+-+
|               |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
### IP header struct [<netinet/ip.h>](https://sites.uclouvain.be/SystInfo/usr/include/netinet/ip.h.html)
```c
# include <netinet/ip.h>
using struct ip
struct ip
  {
    unsigned int ip_hl:4;                 /* header length */           4 bits
    unsigned int ip_v:4;                  /* version */                 4 bits
    u_int8_t ip_tos;                      /* type of service */         1 byte
    u_short ip_len;                       /* total length */            2 bytes
    u_short ip_id;                        /* identification */          2 bytes
    u_short ip_off;                       /* fragment offset field */   2 bytes
    u_int8_t ip_ttl;                      /* time to live */            1 byte
    u_int8_t ip_p;                        /* protocol */                1 byte
    u_short ip_sum;                       /* checksum */                2 bytes
    struct in_addr ip_src, ip_dst;        /* source and dest address */ 4 bytes * 2
  };
```

## ICMP packet
```c
                1               2               3               4
 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8 1 2 3 4 5 6 7 8
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      Type     |      Code     |            Checksum           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                          Message Body                         +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
### ICMP datagram struct [<netinet/ip_icmp.h>](https://sites.uclouvain.be/SystInfo/usr/include/netinet/ip_icmp.h.html)
```c
# include <netinet/ip_icmp.h> 
struct icmp
{
  u_int8_t  icmp_type;        /* type of message, see below */          1 byte
  u_int8_t  icmp_code;        /* type sub code */                       1 byte
  u_int16_t icmp_cksum;       /* ones complement checksum of struct */  2 bytes
  u_int16_t icd_id;                                                     2 bytes
  u_int16_t icd_seq;                                                    2 bytes
  u_int8_t    id_data[1];          16 bytes containing timeval struct
};                                 40 bytes containing payload data
```
# More details

## IP struct fields

#### TTL (time to live)  `u_int8_t ip_ttl`
>Everytime an **IP packet** passes through a **router**, the **time to live field is decremented by 1**. Once it hits **0** the router will **drop the packet** and sends an **ICMP time exceeded message** to the sender. 

*ps : **traceroute** cmd is sending packets incrementing ttl from 1 to n (when we receive a positive response) and resolve time exceed answers adresses. Packets are UDP, TCP or ICMP, but ICMP most used because less blocked by firewalls*

#### IP header length `u_short ip_len`
>The minimum length of an IP header is 20 bytes so with 32 bit increments, you would see value of 5 here. The maximum value we can create with 4 bits is 15 so with 32 bit increments, that would be a header length of 60 bytes. This field is also called the Internet Header Length (IHL).

#### IP id field: `u_int16_t ip_id`
>The Identification field (16 bits) is populated with an ID number unique for the combination of source & destination addresses and Protocol field value of the original packet, allowing the destination to distinguish between the fragments of different packets (from the same source). This does not mean the same ID should be used when fragmenting packets where the source, destination and protocol are the same but that the same ID could be used when they are not.

#### IP offset `u_short ip_off`

>le champ Flag (3 bits) : il permet de gérer la fragmentation :
>- bit 0: réservé – toujours positionné à 0
>- bit 1 : dit bit DF (Don’t Fragment) – S’il est positionné à 0, la fragmentation est autorisée – S’il est positionné à 1 la fragmentation est interdite. Dans ce dernier cas, si le paquet est trop volumineux pour être encapsulé dans une trame, dont le MTU est inférieur à la taille du paquet, la passerelle qui devrait réaliser la fragmentation retournera à l’émetteur du paquet un ICMP « Paquet non fragmentable ».
>- bit 2 : dit bit MF (More Fragment) – S’il est positionné à 0 il indique que le paquet reçu est le dernier du paquet d’origine. S’il est positionné à 1, il indique que le paquet reçu est un fragment du paquet d’origine mais pas le dernier fragment. Un paquet qui n’a pas été fragmenté aura donc toujours ce bit à 0.
>- le champ Fragment Offset : indique la position du premier octet de données du paquet reçu dans la partie donnée du paquet d’origine. Le premier fragment à donc toujours la valeur 0 (position du premier octet), de même que tous paquets non fragment

[more details](https://packetpushers.net/blog/ip-fragmentation-in-detail/#:~:text=The%20Fragment%20Offset%20field%20(13,arrive%20in%20order%20or%20not).)


## ICMP stuff

>The ICMP protocol also does not allow for targeting a specific port on a device.

 When a router or server needs to send an **error message**, the *ICMP packet body or data section*
**always contains a copy of the IP header** of the **packet** that **caused the error.**

### Timestamp stuff [here](https://stackoverflow.com/questions/70175164/icmp-timestamps-added-to-ping-echo-requests-in-linux-how-are-they-represented-t)

Including the **UNIX timestamp** of the time of transmission in the **first data
bytes of the ICMP Echo message payload** is a *trick/optimizatio*n the original 
ping by Mike Muuss used to **avoid keeping track of it locally**.
It exploits the following guarantee made by RFC 792's Echo or Echo Reply Message description:

>*"The data received in the echo message must be returned in the echo reply message."*


## Cyberattacks

### Flood

>A ping flood (ping -f) or ICMP flood is when the attacker attempts to overwhelm a targeted 
device with ICMP echo-request packets. The target has to process and respond to each
packet, consuming its computing resources until legitimate users cannot receive service.

### Ping of death attack

>A ping of death attack is when the attacker sends a ping larger than the maximum allowable 
size for a packet to a targeted machine, causing the machine to freeze or crash. The packet 
gets fragmented on the way to its target, but when the target reassembles the packet into its
 original maximum-exceeding size, the size of the packet causes a buffer overflow.

*The ping of death attack is largely historical at this point. However, older networking 
equipment could still be susceptible to it.*

### Smurf attack

>In a Smurf attack, the attacker sends an ICMP packet with a spoofed source IP address.
 Networking equipment replies to the packet, sending the replies to the spoofed IP and
  flooding the victim with unwanted ICMP packets. Like the 'ping of death,' today the 
  Smurf attack is only possible with legacy equipment.

## nice stuff

### `#define` and array of strings stuff
we cannot make a macro (# define) of an array of strings because macros replace text 
and are not complex dynamic objects. But, we can use a macro containing a list of strings 
to initialize an array of strings.


```c
#define INGREDIENTS_PATES_AU_GRUYERE_LIST {"pâtes", "beurre", "sel", "gruyère"}

int main(int ac, char ** av){
  const char *ingredients[] = INGREDIENTS_PATES_AU_GRUYERE_LIST;
  // by assigning it like this, we have our char** ingredients table
  return 0;
}
```

### Magic numbers 

The term [magic number](https://en.wikipedia.org/wiki/Magic_number_(programming)) or magic constant refers to the **anti-pattern** of using numbers directly in source code.

No magic number = use lots of **define macros** with **explicit names** to make the code clearer.


### Rights qnd capabilities
The capabilities are attached to the process, some are inheritable.
We need to be *root to create raw socket*, we can either run our ping program using sudo `sudo ./program`
or we can **give the rights to our executable** using **setcap** (set capabilities).`

>useful cmds:
>- show binary capabilities: **`grepcap /path/to/the/binary`** (ex: /usr/bin/ping)
>- set capabilities : **`setcap the_capability ./the_binary`** (ex: setcap cap_net_raw=ep ./ft_ping)
>- remove capabilities: **`setcap -r </path/to/binary>`**
>
> *need to be **root or sudo** tu use **setcap***, 
>*setcap create **problems** with **valgrind***



The output of ping from windows machine which by default sends 4 packets and stop.
The output of ping from Linux machine which by default continue pinging until ctrl+c
is pressed to cancel.


comprendre ce que sonr exactement bits frame packets datagram

"Clarify PADDING vs PING frames"
https://www.baeldung.com/cs/networking-packet-fragment-frame-datagram-segment


## compute checksum

 **-> An efficient checksum implementation is critical to good performance.**
  a fraction of a microsecond per data byte can add up to a significant CPU time savings
  overall. 

[compute ICMP checksum](https://forum.microchip.com/s/topic/a5C3l000000LxmDEAS/t221513)
[second link](http://www.faqs.org/rfcs/rfc1071.html)

[compute IP checksum](https://www.thegeekstuff.com/2012/05/ip-header-checksum/)



## compute stddev

*stddev represent the stability of the conenxion.*

It shows how much **variation** there is from the average, or mean. A **low deviation** value indicates 
that the **data points tend to be very close to the mean**, whereas a **high deviation** value indicates 
that **the data are spread out over a large range of values**. A low standard deviation implies that 
there is a more stable, or consistent, performance within the system.
ho to compute stddev:

- Step 1: Calculate the mean. The mean is simply the average of all the response times. This is 
calculated by adding all the response times together and divide by the total number of transactions.
- Step 2: Calculate variance. Variance is calculated by taking each response time and subtracting it 
from the mean. Note that this may end up with a negative number, but each result is squared, so it 
will become a positive number. The last piece is to add up each of the squared values.
- Step 3: Calculate standard deviation. This last step is fairly straightforward. Simply take the total 
of all the squared values from the previous step and find the square root of that value. That will be 
your standard deviation.

[doc how to compute stddev](https://www.dotcom-monitor.com/wiki/knowledge-base/standard-deviation/)

## functions stuff
The `inet_ntoa()` function converts the *Internet host address* in, given in *network byte order*, to a *string 
in IPv4 dotted-decimal notation*. The string is returned in a **statically allocated buffer**, which subsequent 
calls will overwrite. **WHICH SUBSEQUENT CALLS WILL OVERWRITE**.

## socket stuff

`socket()` create an endpoint, and return a file descriptor.

`ip_src` et `ip_dest` sont de type:
```c
  struct in_addr {
      uint32_t       s_addr;     /* address in network byte order */
  };
```
Alors que **getaddrinfo** nous return :
```c
  struct sockaddr {
      sa_family_t     sa_family;      /* Address family */
      char            sa_data[];      /* Socket address */
  };
```
                
**sockaddr** is a generic struct, which is shared by different types 
of sockets. For TCP/IP sockets this struct becomes sockaddr_in (IPv4) or 
sockaddr_in6 (IPv6). For unix sockets it becomes sockaddr_in. Ideally you would use sockaddr_in instead of sockaddr.

But given sockaddr, you could do this to extract IP address from it:
```c
sockaddr foo;

in_addr ip_address = ((sockaddr_in)foo).sin_addr;
//or
in_addr_t ip_address = ((sockaddr_in)foo).sin_addr.s_addr;
```
If you look inside **sockaddr_i**n you will see that the **first 2 byte**s of sa_data are the **port number**. And the **next 4 bytes** are the **IP address**.

>*PS: Please note that the IP address is stored in **network byte order**, so you will probably need to use ntohl (network-to-host) and htonl (host-to-network) to convert to/from host byte order.*



## Network byte order

  The network byte order is defined to always be big-endian, which may differ from the host byte order on a particular machine. Using network byte ordering for data exchanged between hosts allows hosts using different architectures to exchange address information without confusion because of byte ordering. The following C functions allow the application program to switch numbers easily back and forth between the host byte order and network byte order without having to first know what method is used for the host byte order:

- `htonl()` translates an unsigned long integer into network byte order.

- `htons()` translates an unsigned short integer into network byte order.

- `ntohl()` translates an unsigned long integer into host byte order.

- `ntohs()` translates an unsigned short integer into host byte order.

inet_aton() converts the Internet host address cp from the IPv4 numbers-and-dots notation into binary form (in network byte order) and stores it in the structure that inp points to. 


## ping implementation must be from **inetutils-2.0**

[difference between iputils's ping and inetutils's ping](https://unix.stackexchange.com/questions/400351/what-are-the-differences-between-iputils-ping-and-inetutils-ping)

iputils's ping  work only under **linux** unlike inetutils's ping works just as well under **windows**

**iputils's** ping supports quite a few **more feature**s than inetutils's ping, e.g. 
IPv6 (which inetutils implements in a separate binary, ping6), broadcast pings,
 quality of service bits...

iputils's ping supports all the options available on inetutils's ping, so scripts 
written for the latter will work fine with the former. The reverse is not true: 
scripts using iputils-specific options won’t work with inetutils.

As far as why both exist, inetutils is the GNU networking utilities, targeting a
 variety of operating systems and providing lots of different networking tools; 
 iputils is Linux-specific and includes fewer utilities. So typically you’d combine 
 both to obtain complete coverage and support for Linux-specific features, on Linux,
  and only use inetutils on non-Linux systems.

