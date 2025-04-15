# Installation
## 🧱 1. Enable Full JSON Access Logs
Add the following lines to your `Traefik Configuration` @Servers/Configuration file

```
   command:
      ...
        - "--accesslog.filepath=/logs/traefik.log"
        - "--accesslog.format=json"
        - "--accesslog.filters.statusCodes=200-299,400-599"
        - "--accesslog.bufferingSize=0"
        - "--accesslog.fields.headers.defaultMode=drop"
        - "--accesslog.fields.headers.names.User-Agent=keep"
      ...
    volumes:
      ...
         - './logs:/logs'
```

Test if the logs are created. SSH into your machiene and check with after restarting the proxy:

```
nano /opt/traefik-logs/traefik.log
```


## 🔌 2. Enable CrowdSec Bouncer Plugin in Traefik
Add the following lines to your `Traefik Configuration` @Servers/Configuration file.
__Can be also done simoutaniously with step 1.__

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




## 🧾 4. Create Traefik Dynamic Configuration (Middleware)

Add the following lines to your `Traefik Dynamic Configuration` @Servers/Dynamic Configurations/Configuration file

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
          forwardedHeadersTrustedIPs:
            - 10.0.0.0/8
            - 172.16.0.0/12
            - 192.168.0.0/16
          clientTrustedIPs:
            - 10.0.0.0/8
            - 172.16.0.0/12
            - 192.168.0.0/16
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



__Thats it!__