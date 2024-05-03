```mermaid
graph TD
A[ISP #1 ] --> C[Mikrotik роутер]
B[ISP #2 ] --> C[Mikrotik роутер]
C --> D1[HP ElitDesk]
C --> D2[OrangePi 5]
C --> D3[Samsung TV]
C --> D4[LG TV]
C --> D5[SmartHome Aqara GW ]


```

```mermaid
graph TD
A[Wi-Fi Mikrotik роутер] --> B[Server Proxmox]
A[Wi-Fi Mikrotik роутер] --> C[OrangePi]
B --> B1[Print Server]
C --> C1[CasaOS]
B1 --> B2[HP MFP]
C1 --> C2[Pi-hole]
C1 --> C3[Plex Server]
C1 --> C4[Navidrome Server]
C1 --> C5[Uptime-Kuma Server]
C1 --> C6[qBitorrent Server]
C1 --> C7[Prowlarr Server]
```
