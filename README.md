# ft_ping

Mike Muuss wrote this program in December 1983. Its name comes from the sound emitted by a sonar, since their action is similar (emission of a signal which bounces off a target to return to the sender). Subsequently, David L. Mills provided a retroacronym: “Packet InterNet Groper”. `packet network groper? 🤨`

Ping uses the ICMP protocol's mandatory ECHO_REQUEST datagram to elicit an ICMP ECHO_RESPONSE from a host or gateway. ECHO_REQUEST datagrams (``pings'') have an IP and ICMP header, followed by a “struct timeval” and then an arbitrary number of ``pad'' bytes used to fill out the packet.

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
### IP header struct
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
### ICMP datagram struct
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
## `#define` and array of strings stuff
we cannot make a macro (# define) of an array of strings because macros replace text 
and are not complex dynamic objects. But, we can use a macro containing a list of strings 
to initialize an array of strings.
  ex:
```c
#define INGREDIENTS_PATES AU GRUYERE_LIST {"pates", "beurre", "sel", "gruyère"}

int main(int ac, char ** av){
  const char *ingredients[] = INGREDIENTS_PATES_AU_GRUYERE;
  // by assigning it like this, I have my ingredients table
  return 0;
}
```

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

