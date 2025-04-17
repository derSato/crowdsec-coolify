# Installation

## 🧱 1. Enable Full JSON Access Logs

Add the following lines to your `Traefik Configuration` @Servers/Configuration file

```
  volumes:
    ...
      - '/var/log/traefik:/var/log/traefik'
  command:
    ...
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--log.format=json"
      - "--log.level=DEBUG"
      - "--log.maxsize=5000"
      - "--log.maxage=90"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access.log"
      - "--accesslog.format=json"
      - "--accesslog.bufferingsize=100"
      - "--accesslog.fields.headers.defaultmode=keep"
      - "--accesslog.filters.statusCodes=200-299,400-599"




      - "--accesslog.fields.request=keep"
      - "--accesslog.fields.response=keep"
    ...
```

Test if the logs are created. SSH into your machiene and check with after restarting the proxy:

```
nano /opt/traefik-logs/traefik.log
```

## 🔌 2. Enable CrowdSec Bouncer Plugin in Traefik

Add the following lines to your `Traefik Configuration` @Servers/Configuration file. **Can be also done simoutaniously with step 1.**

```
   command:
      ...
        - "--experimental.plugins.crowdsec-bouncer.modulename=github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin"
        - "--experimental.plugins.crowdsec-bouncer.version=v1.4.2"
      ...
```

✅ This makes the plugin available to your middleware configuration.

Restart the Proxy

## 3. Install this Repository as a Docker Compose Project

Point to this directory and install with /docker-compose.yaml

## 4. Activate Dashboaard

WHile checking your logs you see the containers name. it should look something like [Coolify container name]-[Random numbers and letters] Optionally use 'docker ds' to find the container name with ssh. SSH into your machiene and run:

```
docker exec -it [CONTAINER_NAME] cscli bouncers add traefik-bouncer
```

This should return the API ket for 'traefik bouncer'

## 🧾 4. Create Traefik Dynamic Configuration (Middleware)

Now we add middleware to our Dynamic Traefik configuration.

```
http:
  middlewares:
    crowdsec:
      plugin:
        crowdsec-bouncer:
          enabled: true
          crowdsecLapiKey: YOUR_LAPI_KEY
          crowdsecLapiHost: crowdsec:8080
          crowdsecLapiScheme: http
          crowdsecMode: live
          crowdsecAppsecEnabled: true
          crowdsecAppsecHost: crowdsec:7422
          crowdsecAppsecFailureBlock: true
          crowdsecAppsecUnreachableBlock: true
```

🌐 4. Apply Middleware to All HTTPS Services (Globally)

To globally enable CrowdSec for all requests, update your command: in Traefik:

```
- "--entrypoints.https.http.middlewares=crowdsec@file"
```

🧠 This applies the middleware on the https entryPoint — meaning all traffic routed via HTTPS will pass through CrowdSec first.

Want to test on specific routers first instead? You can use labels like:

```
labels:
  - "traefik.http.routers.myservice.middlewares=crowdsec@file"
```

**Thats it!**
