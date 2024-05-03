```mermaid
graph TD
A[Wi-Fi роутер] --> B[Server Proxmox]
A[Wi-Fi роутер] --> D[OrangePi]
B --> C[Print Server]
D --> E[CasaOS]
C --> F[HP MFP]
E --> E1[Pi-hole]
E --> E2[Plex Server]
```
