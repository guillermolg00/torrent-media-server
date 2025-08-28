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
- Para enlazar Radarr/Sonarr con Transmission:
  - Host `transmission`, puerto `9091` (o `localhost:9091`).
  - Usuario/contraseña si configuraste `USER`/`PASS`.
- Con Prowlarr centralizas los indexers y los replicas a Sonarr/Radarr (evita configurarlos en cada app).
