---
stand_alone: true
category: std
submissionType: IETF
ipr: trust200902
lang: en

title: Advertising SaaS Path Performance Metrics using BGP
abbrev: Saas Path Metric
docname: draft-sheng-idr-advertising-saas-path-performance-latest
obsoletes:
updates:
# date: 2023-10-19 -- date is filled in automatically by xml2rfc if not given

area: routing
workgroup: IDR

kw:
  - SDWAN

author:
 -
  ins: C. Sheng
  name: Cheng Sheng
  organization: Huawei
  street: Beiqing Road
  city: Beijing
  country: China
  email: shengcheng@huawei.com
 -
  ins: H. Shi
  name: Hang Shi
  organization: Huawei
  email: shihang9@huawei.com
  role: editor
  street: Beiqing Road
  city: Beijing
  country: China
 -
  ins: L. Dunbar
  name: Linda Dunbar
  organization: Futurewei
  email: linda.dunbar@futurewei.com
  country: United States

--- abstract

This document extends BGP to advertise the SaaS path performance metrics from the gateway sites to branch sites. The user can access SaaS applications through the DIA (Direct Internet Access) link at the branch site or through the DIA link at the gateway site, or use the DIA link of a gateway site for redundancy. This approach will improve the SaaS access experience for end-users.

--- middle

# Introduction {#intro}

With the continuous cloudification of enterprise IT architectures and widespread use of public clouds, more and more enterprises are turning their infrastructures (such as enterprise data centers) to cloudification, abandoning traditional closed IT architectures and using open network architectures.  To further achieve this goal, enterprises' mission-critical applications, such as office, production ERP systems, and sales systems, are migrated to the cloud. In this case, enterprises increasingly rely on software as a service (SaaS) provided by application service providers and prefer to access mission-critical applications from the cloud over the Internet.

Accessing SaaS applications like SalesForce, SharePoint, Dropbox and Office 365 over congested public networks can be unreliable and slow, due to heavy traffic, packet loss, and fluctuating latencies. Application slowness results in poor end-user experience.

This document provides a way to improve the SaaS access experience. As shown in the {{scenario}}, user can access SaaS applications through the DIA (Direct Internet Access) link at the branch site or through the DIA link at the gateway site. The GWs at the gateway site normally have stronger capabilities and will provide SaaS access services for branch sites. The CPE at the branch site need to choose the best path for each SaaS application. The performance of the path between gateway and SaaS application needs to be advertised to CPE. This document extends BGP to advertise the SaaS path performance metrics.

~~~
                               (^^^^^^^^^^^^^^^^^^^^^^^)
                              (       SaaS Apps         )
                              (  +----+  +----+  +----+ )
                              (  |App1|  |App2|  |App3| )
                              (  +----+  +----+  +----+ )
                               (^^^^^^^^^^^^^^^^^^^^^^^)
                                     |   |    |
                                     |   |    |
                                     |  .|----|
                                     | ( |    |)
                                   .-|(  |    | )--.
                            +-----(--+Internet/MPLS )
                           /       '--(  |    | )--'
                          /            ( |    \)
                         /              '|----'\
                        +                | +----|-----------+
                    DIA | Link           \ | +--|--+        |
                        |   +-------------\--| GW2 |        |
                        |  / SD-WAN Tunnel \ +-----+        |
                        | /                |\       Hub Site|
              +----+  +-|/-+ SD-WAN Tunnel | \-----+        |
              |User|--|CPE1|-----------------| GW1 |        |
              +----+  +----+               | +-----+        |
                      Branch Site          +----------------+
~~~
{: #scenario title="SaaS Application Path Performance Optimization Scenario"}

# Terminology

In addition to terms defined in [I-D.ietf-idr-sdwan-edge-discovery], this document uses following terms:

- DIA: Direct Internet Access
- FQDN: Fully Qualified Domain Name
- QoS: Quality of Service
- SaaS: Software-as-a-Service

## Requirements Language

{::boilerplate bcp14-tagged}

# Dynamically Select the Best Path

This section uses the scenario shown in {{scenario}} as an example to describe how to implement the SaaS Path Optimization solution.

Both the Branch and GW routers initiate periodic probes to target SaaS applications. The GW routers advertise the probe result to the Branch routers. The following figure shows the SaaS Path Performance Metrics table on the CPE1. Note that in this example, CPE1, GW1, and GW2 have multiple paths for accessing App1, 2 paths are listed for each device. The access to App2 and App3 is similar, only one entry is listed for the purpose of simplifying the description.

~~~
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |Name|ID|Path |Path Out Intf  |O_QoS|Status| L | D | J | B |F_QoS|
   |    |  |Index|(# Remote)     |     |      |   |   |   |   |     |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I11 |  GE 0/0/1.1   | 75  | Good |  1|150| 40|B01|  75 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I12 |  GE 0/0/1.1   | 80  | Good |  1|160| 40|B01|  80 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I13 |# GW1-System IP| 85  | Good |  0|100| 40|B11|  83 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I14 |# GW1-System IP| 85  | Good |  0|100| 40|B12|  81 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I15 |# GW2-System IP| 90  | Best |  0| 80| 20|B13|  82 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App1|10| I16 |# GW2-System IP| 90  | Best |  0| 80| 20|B14|  88 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App2|20| I02 |  GE 0/0/1.1   | 40  |Issue |  5|180|101|B02|  40 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App2|20| I21 |# GW1-System IP| 80  | Good |  1|100| 70|B21|  75 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App2|20| I22 |# GW2-System IP| 60  | Acct |  3|160| 80|B22|  55 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App3|30| I03 |  GE 0/0/1.1   | 90  | Best |  0| 58| 20|B03|  90 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App3|30| I31 |# GW1-System IP| 80  | Good |  0| 65| 30|B31|  78 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   |App3|30| I32 |# GW2-System IP| 75  | Acct |  2|130| 90|B32|  72 |
   +----+--+-----+---------------+-----+------+---+---+---+---+-----+
   L: Loss          D: Delay
   J: Jitter        B: Bandwidth
   Acct: Acceptable O_QoS: Original QoS
   F_QoS: Final QoS
~~~
{: #Metric-table title="CPE1's SaaS Path Perfermance Metrics Table"}

Upon receiving the QoS score from the GW router, CPE1 will calculates the Final QoS score based on the SD-WAN tunnel status and and the received QoS score. When a user of CPE1 accesses a SaaS applications, CPE1 determines the best performing path toward the SaaS application based on the Final QoS score (F_QoS).

For example If App1 is the target SaaS Application, select the SaaS path that passes through GW2 with the Path Index I16 because it has the highest score: 88. If App2 is the target SaaS Application, select the SaaS path that passes through GW1 with the Path Index I21 because it has the highest score: 75. If App3 is the target SaaS application, select the local SaaS path with the Path Index I03 because it has the highest score: 90.

# The SaaS Path Performance Route

The BGP SD-WAN NLRI as defined in [I-D.ietf-idr-sdwan-edge-discovery] is shown below:

~~~
 +-----------------------------------+
 | Route Type (2 octets)             |
 +-----------------------------------+
 | Length (2 octets)                 |
 +-----------------------------------+
 ~                                   ~
 | Type Specific Value (variable)    |
 ~                                   ~
 +-----------------------------------+
~~~
{: #NLRI  title="BGP SD-WAN NLRI"}

Where:

-  Route (NLRI) Type: 2 octet value to define the encoding of the rest of the SD-WAN NLRI.
-  Length: 2 octets of length expressed in bits as defined in [RFC4760].

This document defines an additional route type to be used for the advertisement of the SaaS Path Performance Metrics between different enterprise sites:

- NLRI Route Type: 2

- Name: SaaS Path Performance Route

## The SaaS Path Performance Route Encoding

~~~
 +--------------------+
 |  Route Type = 2    | 2 octets
 +--------------------+
 |  Length            | 2 octets
 +--------------------+
 |  Site ID           | 4 octets
 +--------------------+
 |  APP ID            | 4 octets
 +--------------------+
 |  APP Req           | 1 octet
 +--------------------+
 |  Path Index Type   | 1 octet
 +--------------------+
 |  Path Index Value  | 3 or 4 or 16 octets
 +--------------------+
 |  SD-WAN-Node-ID    | 4 or 16 octets
 +--------------------+
~~~
{: #Encoding  title="SaaS Path Performance Route"}

Where:

- Route Type: 2, SaaS Path Performance Route
- Length: 2 octets of length expressed in bits as defined in [RFC4760].
- Site ID: 4 octets, A site ID is a unique identifier of an enterprise site in the SD-WAN network.
- APP ID: 4 octets, SaaS Application ID, a unique Application ID to identify different applications. Application may be deployed using different IP address in different area. Thus an ID is needed to identify the application.
- APP Req: 1 octet, Application requirement to indicate the application requirement of the path quality. For example, an real time video conferencing application requires higher quality than a background file backup application. The value includes:
  - Type = 1: default;
  - Type = 2: Medium;
  - Type = 3: High;
- Path Index Type: Indicates the type of the path index.
- Path Index Value: a Path Index Type specific Value:
  - Type 1, the Path Index Value is a 4-byte local index value, which is used to identify an outbound interface for accessing SaaS applications.
  - Type 2, the Path Index Value is a 3-byte MPLS label, which is used to identify an outbound interface for accessing the SaaS application.
  - Type 3, The Path Index Value is a 16-byte SRv6 SID, which is used to identify an outbound interface for accessing a SaaS application, and its Endpoint Behavior is End.DT2SaaSPath: Decapsulate SRv6 packet, then send the packet to the target SaaS application from the outbound interface indicated by the SRv6 SID.
- SD-WAN Node ID: The node's IPv4 or IPv6 address.

## The SaaS Path Performance Metrics Encoding

The Metadata Path Attribute has been as defined in [I-D.ietf-idr-5g-edge-service-metadata]. This document introduces some additional Sub-TLVs to encode the SaaS Path Performance Metrics and SaaS Application Information.

Another option is to use the above Sub-TLVs in the Tunnel Encapsulation Attribute [RFC9012].  In this option, the tunnel type "SaaS Application Path Performance" is added.

### The SaaS Path Delay Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |    Delay Sub-Type = TBD1      |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |         Delay                 |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #Path-Delay  title="SaaS Path Delay Sub-TLV"}

Where:

* Delay Sub-Type: TBD by IANA.
* Length: 2 octets, the total number of octets of the value field.
* Delay: 2 octets, this field indicates the packet transmission delay, in milliseconds.

### The SaaS Path Loss Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |    Loss  Sub-Type = TBD2      |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |     Loss      |
     +-+-+-+-+-+-+-+-+
~~~
{: #Path-Loss  title="SaaS Path Loss Sub-TLV"}

Where:

*  Loss Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Loss: 1 octet, this field indicates the packet loss rate (%).

### The SaaS Path Jitter Sub-TLV format

~~~
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |    Jitter Sub-Type = TBD3     |               Length          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |         Jitter                |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #Path-Jitter  title="SaaS Path Jitter Sub-TLV"}

Where:

*  Jitter Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Jitter: 2 octets, this field indicates the jitter on the SaaS Path. Range: 1 through 1000 milliseconds

###  The SaaS Path Bandwidth Sub-TLV format

~~~
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Bandwidth Sub-Type = TBD4    |               Length          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           Bandwidth                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #Path-Bandwidth  title="SaaS Path Bandwidth Sub-TLV"}

Where:

*  Bandwidth Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Bandwidth: 4 octets, this field indicates the bandwidth of the SaaS Path.

### The SaaS Path Status Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |    Status Sub-Type = TBD5     |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |    Status     |
     +-+-+-+-+-+-+-+-+
~~~
{: #Path-Status title="SaaS Path Status Sub-TLV"}

Where:

*  Status Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Status: 1 octet, Network assessment, there are 6 levels as
   follows:
   -  100: Best
   -  80: Good, Meets recommendations
   -  60: Acceptable
   -  40: Users may experience issues
   -  20: Users may complain
   -  0: Network problems

### The SaaS Path QoS Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |    QoS Sub-Type = TBD6        |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |      QoS      |
     +-+-+-+-+-+-+-+-+
~~~
{: #Path-QoS title="SaaS Path QoS Sub-TLV"}

Where:

*  QoS Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  QoS: 1 octet, Quality of Service, 1-100, with 1 being the worst, and 100 being the best. The QoS value is calculated based on the values of Loss, Jitter, Delay, and Status.

### The SaaS Application Name Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | SaaS AppName Sub-Type = TBD7  |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     ~                                                               ~
     |             Application Name (1-n Octets)                     |
     ~                                                               ~
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #App-Name title="Saas Application Name Sub-TLV"}

Where:

*  SaaS AppName Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Application Name: The name of the application represented as a string, such as Salesforce, Dropbox, Office 365, and so on.

### The SaaS Application Domain Name Sub-TLV format

~~~
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     | AppDomainName Sub-Type = TBD8 |               Length          |
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     ~                                                               ~
     |         Application Domain Name (Variable)                    |
     ~                                                               ~
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
{: #App-Domain-Name title="SaaS Application Domain Name Sub-TLV"}

Where:

*  AppDomainName Sub-Type: TBD by IANA
*  Length: 2 octets, the total number of octets of the value field.
*  Application Domain Name: The domain name of the application represented as a string, such as www.salesforce.com, www.baidu.com, www.iana.org, www.dropbox.com, www.microsoft.com, and so on.

# Security Considerations

TBD.

# IANA Considerations

TBD.

--- back

# Contributors

Shunwan Zhuang
Huawei
Email: zhuangshunwan@huawei.com

Penghe Tang
Huawei Technologies
Email: tangpenghe@huawei.com@huawei.com
