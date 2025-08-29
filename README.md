# Media Stack (Radarr + Sonarr + Plex)

Este stack docker levanta:
- Radarr (películas) → http://localhost:7878
- Sonarr (series) → http://localhost:8989
- Prowlarr (indexers) → http://localhost:9696
- Plex (media server) → http://localhost:32400/web
- Transmission (cliente BT) → http://localhost:9091

---

## Estructura de carpetas

```
T-media/
├── docker-compose.yml
├── .env
├── README.md
├── config/
│   ├── radarr/
│   ├── sonarr/
│   ├── prowlarr/
│   ├── plex/
│   └── transmission/
├── data/
│   ├── movies/
│   └── tv/
└── downloads/
    ├── complete/
    ├── incomplete/
    └── watch/
```

- `config/*`: configuraciones persistentes de cada app.
- `data/movies` y `data/tv`: tus bibliotecas.
- `downloads/*`: ruta de descargas que Radarr/Sonarr compartido con transmission

## Variables (.env)

- `PUID`/`PGID`: tu usuario/grupo local para permisos (ya tomado de tu sistema).
- `TZ`: zona horaria.
- `PLEX_CLAIM`: token opcional para reclamar Plex.

### VPN (WireGuard con Gluetun)
- `VPN_SERVICE_PROVIDER`: por ejemplo `mullvad`, `protonvpn` o `custom`.
- `VPN_TYPE`: dejar en `wireguard`.
- `WIREGUARD_PRIVATE_KEY` y `WIREGUARD_ADDRESSES`: del perfil WireGuard.
- Si usas `custom`: también `WIREGUARD_PUBLIC_KEY`, `WIREGUARD_ENDPOINT_IP`, `WIREGUARD_ENDPOINT_PORT`.
- `FIREWALL_OUTBOUND_SUBNETS`: subred(es) LAN permitidas (ej. `192.168.0.0/16`).


## Primeros pasos en las apps

- Prowlarr
  - URL: http://localhost:9696
  - Añade tus indexers (Torznab/NZB/privados) en Settings → Indexers.
  - En Settings → Apps: añade Sonarr y Radarr con estas URLs internas y API keys:
    - Sonarr: URL `http://sonarr:8989`, API Key desde Sonarr → Settings → General.
    - Radarr: URL `http://radarr:7878`, API Key desde Radarr → Settings → General.
  - Guarda y pulsa “Sync Indexers” para empujarlos a Sonarr/Radarr.

- Radarr
  - URL: http://localhost:7878
  - Root folder: `/movies`
  - Carpeta de descargas (si la usas): `/downloads`
  
- Sonarr
  - URL: http://localhost:8989
  - Root folder: `/tv`
  - Carpeta de descargas (si la usas): `/downloads`

- Transmission
  - URL: http://localhost:9091
  - Preferencias → Ubicaciones:
    - Complete: `/downloads/complete`
    - Incomplete: `/downloads/incomplete` (opcional)
  - La carpeta `/watch` admite .torrent automáticos.

- Plex
  - URL: http://localhost:32400/web
  - Si tienes `PLEX_CLAIM`, el alta inicial es más fácil.
  - Agrega bibliotecas apuntando a `/data/movies` y `/data/tv`.

## Notas

- Radarr/Sonarr y Transmission comparten `/downloads`, por lo que NO necesitas Remote Path Mapping.
- Para enlazar Radarr/Sonarr con Transmission cuando TODO está detrás de la VPN:
  - En Radarr/Sonarr, cliente de descargas Transmission: URL `http://localhost:9091` (comparten stack de red con Gluetun).
  - Desde tu host (navegador): http://localhost:9091.
  - Usuario/contraseña si configuraste `USER`/`PASS`.
- Con Prowlarr centralizas los indexers y los replicas a Sonarr/Radarr (evita configurarlos en cada app).

## VPN: todo el tráfico del stack por WireGuard

Este stack incluye un cliente VPN (`gluetun`) que levanta un túnel WireGuard y enruta TODO el tráfico de Transmission, Prowlarr, Radarr, Sonarr y Plex a través de él. Los puertos de todas estas apps se publican desde el contenedor `gluetun` (9091, 9696, 7878, 8989, 32400, etc.).

Pasos:
- Consigue las credenciales/perfil WireGuard de tu proveedor (claves y endpoint).
- Rellena las variables en `.env` (sección VPN).
- Levanta el stack: `docker compose up -d`.

Notas:
- Dentro de Prowlarr (Settings → Apps), usa estas URLs: (tuve que usar 127.0.0.1, porque localhost fallaba el check de ipv6)
  - Radarr: `http://localhost:7878`
  - Sonarr: `http://localhost:8989`
  - Transmission: `http://localhost:9091`
  - Instance URL de Prowlarr: `http://localhost:9696`
- Desde tu host, accedes igual por `http://localhost:<puerto>`.
- Si detectas problemas de descubrimiento DLNA de Plex tras la VPN, puedes desactivarlo o usar acceso directo por `http://localhost:32400/web`.
