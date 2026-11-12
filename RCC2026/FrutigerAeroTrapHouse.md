# Frutiger Aero Trap House

RowdyCyberCon CTF 2026 · OSINT · Solved

## Challenge

The challenge provided a Wi-Fi network name (SSID): "Frutiger Aero Trap House". The goal was to find the city where that network is located.

## My approach

Wireless access points are constantly broadcasting their SSIDs, and **WiGLE** (Wireless Geographic Logging Engine) is a crowdsourced database of hundreds of millions of these broadcasts, each tied to the GPS coordinates where a wardriver observed it. An unusual, specific SSID like this one is effectively a unique fingerprint — so if anyone has ever logged it, WiGLE can geolocate it.

## Query

WiGLE exposes a REST API. Searching for the SSID (using `%` wildcards around the distinctive words to tolerate spacing and capitalization):

```bash
curl -s -u "AID...:TOKEN..." \
  "https://api.wigle.net/api/v2/network/search?ssidlike=%25aero%25trap%25&resultsPerPage=25"
```

## Result

The search returned two radios (a 2.4 GHz and a 5 GHz band) belonging to a single WPA3 router — which is exactly what one dual-band home router looks like in the data. Both were located in:

- Denver, Colorado, US (Knox Court, ZIP 80219; approximate coordinates 39.7223, -105.0325).

The two bands sharing a location confirmed it was one physical device, not two coincidental matches.

Flag: the precise coordinates

## Takeaway

A Wi-Fi name is not private, it's a public broadcast, and a distinctive one is a searchable identifier. The same crowdsourced wardriving data that security researchers use to map wireless coverage also means any sufficiently unique SSID can be tied back to a physical address. Cross-checking the two radio bands against each other was the verification step that turned a hit into a confident answer.
