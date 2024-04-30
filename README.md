# traefik-tls
Configuration to use traefik as TLS Reverse Proxy for multiple container app

1. Clone this repo
2. `docker compose up -d`
3. Check the name of the created network bridge: `docker network ls`, usually its: _networkname-folderprojectname_ (in my usecase _traefik-global_traefiknet_)
4. Prepare the private key/certificate pair for your target container app (use OpenSSL or your Private PKI)
5. Follow Steps 1 to 5 to generate your self signed cert with SAN attribute: https://github.com/s0p4L1n3/Graylog-Ready-to-go-Compose#initial-step
6. Copy the certificate and key file to `./certs`
7. Edit `traefik.yml` and add
```
certificates:
    - certFile: /certs/dnsname.lab.lan-crt.pem
      keyFile: /certs/dnsname.lab.lan-key.pem
````


In your app container compose file you just need to add the labels below:

For example if you service is named: webapp1

```
labels:
  - "traefik.enable=true"
  - "traefik.http.routers..rule=Host(`dnsname.lab.lan`)"
  - "traefik.http.routers.webapp1.entrypoints=websecure"
  - "traefik.http.routers.webapp1.tls=true"
  - "traefik.http.services.webapp1.loadbalancer.server.port=80"
  - "traefik.docker.network=traefik-global_traefiknet"
```

You are telling your app container to use Traefik router directive. Then when you will access `https://dns.lab.lan`, Traefik will redirect (but internally) your traffic to the target container on port 80.
It will use then the certificate and private key to secure and encrypt the communication between you (client) and traefik (server).


   
