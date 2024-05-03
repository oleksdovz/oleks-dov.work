
```mermaid
graph LR
  subgraph Internet
    A[ISP #1]
    B[ISP #2]
  end

  subgraph Local Network
    C[MikroTik router]
    D[HP ElitDesk PC]
    E[OrangePi SBC]
    F[Samsung TV]
    G[LG TV]
    H[SmartHome 'Aqara GW']
  end

  A --> C
  B --> C
  C --> D
  C --> E
  C --> F
  C --> G
  C --> H
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
