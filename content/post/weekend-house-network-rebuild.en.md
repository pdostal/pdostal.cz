---
title: Weekend house network rebuild
date: 2025-06-13
tags:
  - network
---

A few weeks ago, my friend Ondra and I performed a significant refresh of our weekend house network.

The objectives were:

1. Connect two buildings with a fiber link
2. Install an LTE dish to provide better Internet connectivity

The further objectives are:

3. Replace the old SSIDs with a single SSID to enable seamless roaming across the property
4. Consolidate two separate networks and segment the one newly created network into multiple VLANs and reassign the existing networking and IoT devices
5. Set up new firewall rules to secure the infrastructure and allow only the necessary traffic

<!--more-->

## The fiber link
The two buildings that we want to connect are only 5 meters apart, so we used a ready-made single-mode two-core outdoor-rated armored fiber patch. We have installed an anchor on each end of the fiber span to hold it in place and gave it some tolerance to allow the cable to move. We installed it as high as possible, making it nearly invisible and safe from tall vehicles. We have achieved a very low loss of just around 1 dBm on the whole fiber run, which means that we did not damage the fiber cores nor make the connectors dirty.

In the small building, the cable runs comfortably around the walls to the `Mikrotik SR-10U` rack, where it's connected to the `Mikrotik RB5009` main router called "Kadovánek" after the tasty local herb liqueur. There is enough space to comfortably service the equipment, plus the main LTE uplink is also located here.

In the bigger building, the cable is attached to a wooden roof beam and later fished through a conduit to a small in-wall utility box where the Mikrotik CRS112 and CSS610 are located and blinking happily.

### Alternatives
Before deciding on a fiber link between the buildings, we have also considered using a wireless alternative. It would be easier to install, setup and wouldn't require drilling into the houses's front walls and pulling 30 meters of fiber through tight spaces and lofts.  
Even with all the work required to link the buildings with fiber, we decided to go for it. We want the stability, speed (10 Gbps in our case) and especially future upgradability. We could just swap out the optic transceivers when we want to.

![Attic](/2506_optika_nova.jpeg)
![Anchor](/2506_kotvicka.jpeg)
![Rack on the small building](/2506_rack.jpeg)

## Internet connectivity
Previously, we were connected by 5GHz ac link from a locally present big ISP, but due to our remote location, the maximum speed they offer for end users is 30/3 Mbps. The whole village is connected by a microwave link (Racom Ray3 radio made in 🇨🇿), so we were hunting for other options.  
There were three options:

1) Business connectivity from this ISP via a dedicated microwave or milimeterwave PtP connection: This would be the ideal solution with an SLA and plenty of speed, but is roughly seven times more expensive.
2) Starlink: I considered Starlink and I'm glad I didn't choose it, as it would be three times slower and three times more expensive than the solution we eventually chose.
3) LTE: We opted for LTE as it was something we could try and either install permanently or return. It was quite risky as the cell coverage in our locality is very poor (measured from cellphone) and providers don't offer "wireless Internet service" in our area.

![LHG LTE18 antena](/2506_lhg_lte18.jpeg)
![LTE signal](/2506_lte_signal.jpeg)

### Obtaining a SIM card for the LTE antenna

A month ago, I was approached by an external call center representing T-Mobile. They offered me a decent price for an unlimited data SIM card. So when I decided to purchase it, I called T-Mobile and received another, this time two times more expensive, offer. The representative was quickly done with me, and as I already have a business account with T-Mobile with several high-tier services, I was quite disappointed.

Luckily, I made a third attempt to get the SIM card and this time I got lucky. Instead of calling, I just visited the T-Mobile branch at Nový Smíchov shopping mall, where very pleasant Valerie took her time to learn my account and together we discussed different options. In the end, I received my SIM card with unlimited speed and unlimited data for a very decent price.

We knew from [GSMWeb.cz](https://gsmweb.cz/) (a very cool user-contributed BTS mapping project) that there is a radio tower on Buchtův kopec - it's a big one, so it most probably has backup power and is connected by fiber. It is 5 kilometers away and we do have a line of sight from the rooftop to it. Just barely above another hill, but the Fresnel zone should be just fine for bands that we are going to use. After a successful first connection attempt to the tower from the ground, the antenna was mounted on the rooftop. We get a solid ~350 Mbps down and ~50 Mbps up, the latency is worse at ~50 ms. The signal seems so far very solid; we measure RSSI at ~46 dBm and RSRP at ~75 dBm. Let's wait for what rain, storms, and winter bring.

## Single main SSID

Before, when a client connected to one SSID and walked too far from the corresponding building, the connection dropped. As the buildings are close to each other and there is a lot of outdoor stuff to do, the wireless experience was not very reliable. After we merged the networks together, clients can seamlessly roam around the properties and enjoy a stable connection. The Mikrotik CapsMan supports roaming so there are no reconnects. There is a total of eight 802.11ax WiFi access points covering the whole property and all of its grounds including patios, gardens, parking spots and more.

## Network segmentation

There used to be two separate networks with two separate routers. In those networks clients were mixed together with IoT devices and everyone had access everywhere. In order to increase the security, the one big layer 2 network was split by creating three networks, two of them being inside of a VLAN:

1) Public network: This is the main, untagged, network where all the clients connect. The network is accessible via the main SSID mentioned above.
2) IoT network: A new IoT network where all the sensors for monitoring electricity and air quality are. The network is also accessible via a separate SSID.
3) The management network: There are all the access points and switches. There are no users and the network is not accessible wirelessly.

## Firewall

Access to the Internet is allowed from all three networks but both IoT as well as Public networks are isolated with the following exceptions:

* The router is accessible from the Public network via SSH for configuration purposes.
* Our Home Assistant box is accessible from the Public network via HTTPS.

## Conclusion

We think we did a lot of great physical as well as later configurational work. The network is now more reliable, and more secure and the Internet connectivity is much faster than before.  
There is still a lot of room for progress but changes are risky as the village is pretty remote.


**Note from Ondra**
> I had a great time. I'm a big network enthusiast and enjoyed my work with Pavel very much and I am glad that I could help him. We already have ideas about what to do next time we are there, such as 5G uplink instead of LTE.
