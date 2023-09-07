# Projector-DDoS

Performs basic DDoS attacks against simple web application

# Run
```shell
docker-compose up -d
```

# Stop
```shell
docker-compose down
```

# Change DDoS attack

Inside [docker-compose](./docker-compose.yml) uncomment 1 of the commands for `attacker` container

# Results

## HTTP flood:
```
HPING website (eth0 172.25.0.10): NO FLAGS are set, 40 headers + 0 data bytes
2023-09-07T18:26:19.232113241Z hping in flood mode, no replies will be shown
2023-09-07T18:28:23.173275691Z 
2023-09-07T18:28:23.173352042Z --- website hping statistic ---
2023-09-07T18:28:23.173360901Z 18442975 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:28:23.173363965Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```
## TCP SYN flood
```
HPING website (eth0 172.25.0.10): S set, 40 headers + 0 data bytes
2023-09-07T18:29:38.321013489Z hping in flood mode, no replies will be shown
2023-09-07T18:29:48.759676567Z 
2023-09-07T18:29:48.759709074Z --- website hping statistic ---
2023-09-07T18:29:48.759713335Z 485067 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:29:48.759716106Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```

## UDP flood:
```
HPING website (eth0 172.25.0.10): udp mode set, 28 headers + 0 data bytes
2023-09-07T18:30:04.023196175Z hping in flood mode, no replies will be shown
2023-09-07T18:30:13.497361915Z 
2023-09-07T18:30:13.497415685Z --- website hping statistic ---
2023-09-07T18:30:13.497424164Z 528489 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:30:13.497427320Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```
## TCP FIN Flood
```
hping in flood mode, no replies will be shown
2023-09-07T18:30:28.736356504Z HPING website (eth0 172.25.0.10): F set, 40 headers + 0 data bytes
2023-09-07T18:30:36.724180103Z 
2023-09-07T18:30:36.724215613Z --- website hping statistic ---
2023-09-07T18:30:36.724219611Z 865471 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:30:36.724222088Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```

## TCP RST Flood
```
HPING website (eth0 172.25.0.10): R set, 40 headers + 0 data bytes
2023-09-07T18:30:50.048143916Z hping in flood mode, no replies will be shown
2023-09-07T18:30:57.962449298Z 
2023-09-07T18:30:57.962477761Z --- website hping statistic ---
2023-09-07T18:30:57.962481584Z 1143846 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:30:57.962484098Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```

## PUSH and ACK Flood
```
HPING website (eth0 172.25.0.10): AP set, 40 headers + 0 data bytes
2023-09-07T18:31:11.784955546Z hping in flood mode, no replies will be shown
2023-09-07T18:31:20.891514673Z 
2023-09-07T18:31:20.891545230Z --- website hping statistic ---
2023-09-07T18:31:20.891548954Z 489763 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:31:20.891551405Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```

## ICMP Flood
```
using eth0, addr: 172.25.0.20, MTU: 1500
2023-09-07T18:31:35.661355688Z hping in flood mode, no replies will be shown
2023-09-07T18:31:43.321250302Z HPING website (eth0 172.25.0.10): icmp mode set, 28 headers + 0 data bytes
2023-09-07T18:31:43.321266458Z 
2023-09-07T18:31:43.321334038Z --- website hping statistic ---
2023-09-07T18:31:43.321338874Z 371511 packets tramitted, 0 packets received, 100% packet loss
2023-09-07T18:31:43.321341413Z round-trip min/avg/max = 0.0/0.0/0.0 ms
```

# DDoS Protection

## Limit Connections:
Using http module to limit the number of connections a client can open:

```nginx configuration
    server {
        location / {
            limit_conn conn_limit_per_ip 10; # Adjust this number as needed
        }
    }
```

## Limit Request Rates:
Limit the rate at which requests are accepted from a single IP:

```nginx configuration
    server {
        location / {
            limit_req zone=req_limit_per_ip burst=10; # Allow bursts of 10 requests
        }
    }
```

## Drop Invalid Requests:
If we receive invalid requests, we can drop them:

```nginx configuration
if ($request_method !~ ^(GET|HEAD|POST)$ ) {
return 444;
}
```

## Timeout Settings:
Adjust connection timeout settings to quickly free up resources from slow clients:

```nginx configuration
client_body_timeout 10;
client_header_timeout 10;
keepalive_timeout 5 5;
send_timeout 10;
```


## GeoIP Blocking:
If you notice malicious traffic predominantly coming from certain countries, 
and your service isn't targeted at those regions, you can use the GeoIP module to block traffic from specific countries.


## Hide Nginx Version:

```nginx configuration
server_tokens off;
```

## Use CDNs and DDoS Protection Services:
Content Delivery Networks (CDNs) like Cloudflare, Akamai, or AWS Shield can help absorb DDoS traffic and keep your server from becoming overwhelmed.

## Upstream Timeouts:
If you're using Nginx as a reverse proxy, configure upstream timeouts:

```nginx configuration
proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;
```