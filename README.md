# Installation

## 1. Enable Full JSON Access Logs

Add the following lines to your `Traefik Configuration` @Servers/Configuration file

```
  volumes:
    ...
      - '/var/log/traefik:/var/log/traefik'
  command:
    ...
      - "--experimental.plugins.crowdsec-bouncer.modulename=github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin"
      - "--experimental.plugins.crowdsec-bouncer.version=v1.4.2"
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--log.format=json"
      - "--log.level=INFO"
      - "--log.maxsize=5000"
      - "--log.maxage=90"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access.log"
      - "--accesslog.format=json"
      - "--accesslog.bufferingsize=100"
      - "--accesslog.fields.headers.defaultmode=keep"
      - "--accesslog.filters.statusCodes=200-299,400-599"



      <!-- For APPSEC -->
      - "--accesslog.fields.request=keep"
      - "--accesslog.fields.response=keep"
    ...
```

Test if the logs are created after restarting the proxie. SSH into your machiene and check with after restarting the proxy:

```
cd /var/log/traefik
ls
nano traefik.log
nano access.log
```

## 2. Install this Repository as a Docker Compose Project

Point to this directory and deplot it with /docker-compose.yaml.

## 3. Activate Dashboaard

While checking your logs you see the new installed containers name. it should look something like [Crowdsec container name]-[Random numbers and letters] Optionally use 'docker ps' to find the container name with ssh. SSH into your machiene and run:

```
docker exec -it [CONTAINER_NAME] cscli bouncers add traefik-bouncer
```

This returns the API key for 'crowdsecs api'

## 4. Enable CrowdSec Bouncer within the Dynamic Configuration (Middleware)

Create a new File called 'crowdsec.yaml' in the 'Middleware' section of your Dynamic Configuration (Middleware)

```
http:
  middlewares:
    crowdsec:
      plugin:
        crowdsec-bouncer:
          enabled: true
          crowdsecLapiKey: YOUR_API_KEY
          crowdsecAppsecEnabled: true
          crowdsecAppsecHost: 'crowdsec:7422'
```

## 5. Apply Middleware to All HTTPS Services (Globally)

To globally enable CrowdSec for all requests, update your command: in Traefik:

```
    command:
      - "--entrypoints.https.http.middlewares=crowdsec@file,ratelimit@file"
```

🧠 This applies the middleware on the https entryPoint — meaning all traffic routed via HTTPS will pass through CrowdSec first.

Want to test on specific routers first instead? You can use labels like:

```
    labels:
      - "traefik.http.routers.myservice.middlewares=crowdsec-bouncer@file"
```

## 6. Test the configurtation

Like before access the crowdsec cli thru the cordsc container:

```
docker exec -it [CONTAINER_NAME] cscli decisions add --ip [YOUR IP]
```

Connect to any exposed URL and check if the bouncer worked. Also the following command shows if all the connections are handled by appsec after viewing any https page

```
docker exec -it [CONTAINER_NAME] cscli metrics show appsec
```

Respectively for crowdsec

```
docker exec -it [CONTAINER_NAME] cscli decisions list
```

## 7. Front Security

If you are using any service in front like cloudflare. add these to trusted ips so they dont get flagged as ddos:

Traefik.yaml

```
    command:
    ...
      - "--entryPoints.https.forwardedHeaders.trustedIPs=173.245.48.0/20,103.21.244.0/22,103.22.200.0/22,103.31.4.0/22,141.101.64.0/18,108.162.192.0/18,190.93.240.0/20,188.114.96.0/20,197.234.240.0/22,198.41.128.0/17,162.158.0.0/15,104.16.0.0/13,104.24.0.0/14,172.64.0.0/13,131.0.72.0/22,2400:cb00::/32,2606:4700::/32,2803:f800::/32,2405:b500::/32,2405:8100::/32,2a06:98c0::/29,2c0f:f248::/32"
```

Cloudflares list can be found at [https://www.cloudflare.com/ips/](https://www.cloudflare.com/ips/)

**Thats it!**

## Bonus: Rate Limiter

Inside Dynamic Configuration (Middleware) add a new File:

```
http:
  middlewares:
    ratelimit:
      rateLimit:
        average: 100
        burst: 200
```

Inside traefik.yaml

```
    command:
      - "--entrypoints.https.http.middlewares=crowdsec@file,ratelimit@file"  # This Replaces the line of step 5
      - "--entrypoints.http.http.middlewares=ratelimit@file"
```
