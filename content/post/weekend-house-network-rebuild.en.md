---
title: Weekend house network rebuild
date: 2024-06-13
tags:
  - network
---

A few weeks ago, my friend Ondra and I performed a significant refresh of our weekend house network.

The objectives were:
1. Connect two buildings with a fibre link
2. Install an LTE dish to provide better Internet connectivity

The further objectives are:
4. Replace the old SSIDs with a single SSID to enable seamless roaming across the property
5. Segment the network into multiple VLANs and reassign the existing networking and IoT devices
6. Set up new firewall rules to secure the infrastructure and allow only the necessary traffic

<!--more-->

## The fibre link
The buildings are only 5 meters apart, so we used a ready-made single-mode armored cable. There are anchors on each wall to allow the cable some space to move. We installed it as high as possible, making it nearly invisible and safe from tall vehicles.

In the small building, the cable runs comfortably around the walls to the Mikrotik SR-10U rack, where it's connected to the Mikrotik RB5009 main router. There is enough space to comfortably service the equipment, plus the main LTE uplink is also located here.

In the bigger building, the cable is attached to the beam and later fished through a conduit to a small in-wall utility box where the Mikrotik CRS112 and CSS610 are located.

## Internet connectivity
Previously, we were connected by 5GHz ac link from locally present big ISP, but due to our remote location, the maximum speed they offer for end users is 30/3 Mbps. The whole village is connected by a 60G link, so we were hunting for other options, and so far there are three:

1) Business connectivity from this ISP via dedicated 60G PtP connection: This would be the ideal solution but is roughly seven times more expensive.
2) Starlink: I considered Starlink and I'm glad I didn't choose it, as it would be three times slower and three times more expensive than the solution we eventually chose.
3) LTE: We opted for LTE as it was something we could try and either install permanently or return. It was quite risky as the cell coverage in our locality is very poor (measured from cellphone) and providers don't offer "wireless Internet service" in our area.

### Obtaining a SIM card for the LTE antenna

A month ago, I was approached by an external call center representing T-Mobile. They offered me a decent price for an unlimited data SIM card. So when I decided to purchase it, I called T-Mobile and received another, this time two times more expensive, offer. The representative was quickly done with me, and as I already have a business account with T-Mobile with several high-tier services, I was quite disappointed.

Luckily, I made a third attempt to get the SIM card and this time I got lucky. Instead of calling, I just visited the T-Mobile branch at Nový Smíchov shopping mall, where very pleasant Valerie took her time to learn my account and together we discussed different options. In the end, I received my SIM card with unlimited speed and unlimited data for a very decent price.

We knew from GSMWeb.cz that there is a radio tower on Buchtův kopec - it's a big one, so it most probably has backup power and is connected by fiber. It is 5 kilometers away and we do have a line of sight from the rooftop to it. After a successful first attempt from the ground, the antenna was mounted on the rooftop. We get a solid ~350 Mbps down and ~50 Mbps up, the latency is worse at ~50 ms. The signal seems so far very solid; we measure RSSI at ~46 dBm and RSRP at ~75 dBm. Let's wait what rain, storms, and winter bring.

## Single main SSID

Before, when a client connected to one SSID and walked too far from the corresponding building, the connection dropped. As the buildings are close to each other and there is a lot of outdoor stuff to do, the wireless experience was not very reliable. After we merged the networks together, clients can seamlessly roam around the properties and enjoy a stable connection. The Mikrotik CapsMan supports roaming so there are no reconnects. The patios, gardens, as well as parking lots, are covered by already installed outdoor access points.

## Network segmentation

There used to be two separate networks with two separate routers. In those networks clients were mixed together with IoT devices and everyone had access everywhere. To increase the security three new networks were created:

1) Public network: This is the main, untagged, network where all the clients connect. The network is accessible via the main SSID mentioned above.
2) IoT network: A new IoT network where all the sensors for monitoring electricity and air quality are. The network is also accessible via separate SSID.
3) The management network: There are all the access points and switches. There are no users and the network is not accessible wirelessly.

## Firewall

Access to the Internet is allowed from all three networks but both IoT as well as Public networks are isolated with the following exceptions:

* The router is accessible from the Public network via SSH for configuration purposes.
* Our Home Assistant box is accessible from the Public network via HTTPS.

## Conclusion

We think we did a lot of great physical as well as later configurational work. The network is now more reliable and more secure and the Internet connectivity is faster. There is still a lot of room for progress but changes are risky as the village is pretty remote.
