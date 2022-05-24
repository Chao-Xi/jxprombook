# **3 Reverse proxies and load balancers**

### **[1 Nginx](./4Nginx.md)**

1. **HTTP errors 4xx**
2. **HTTP errors 5xx**
3. **Nginx latency high**

### **[2 Apache](./12Apache.md)**

1. **Apache down**
2. **Apache workers load**
3. **Apache restart**

### **[3 HaProxy](./13HaProxy.md)**

**Embedded exporter (HAProxy >= v2)**

1. **HAProxy high HTTP 4xx error rate backend**
2. **HAProxy high HTTP 5xx error rate backend**
3. **HAProxy high HTTP 4xx error rate server**
4. **HAProxy high HTTP 5xx error rate server**
5. **HAProxy server response errors**
6. **HAProxy backend connection errors**
7. **HAProxy server connection errors**
8. **HAProxy backend max active session > 80%**
9. **HAProxy pending requests**
10. **HAProxy HTTP slowing down**
11. **HAProxy retry high**
12. **HAproxy has no alive backends**
13. **HAProxy frontend security blocked requests**
14. **HAProxy server healthcheck failure**

**prometheus/haproxy_exporter (HAProxy < v2)**

1. **HAProxy down**
2. **HAProxy high HTTP 4xx error rate backend**
3. **HAProxy high HTTP 5xx error rate backend**
4. **HAProxy high HTTP 4xx error rate server**
5. **HAProxy high HTTP 5xx error rate server**
6. **HAProxy server response errors**
7. **HAProxy backend connection errors**
8. **HAProxy server connection errors**
9. **HAProxy backend max active session**
10. **HAProxy pending requests**
11. **HAProxy HTTP slowing down**
12. **HAProxy retry high**
13. **HAProxy backend down**
14. **HAProxy server down**
15. **HAProxy frontend security blocked requests**
16. **HAProxy server healthcheck failure**

### **[4 Traefik](./14Traefikv1.md)**

**ExporterV2: observability/metrics/prometheus/**

1. **Traefik service down**
2. **Traefik high HTTP 4xx error rate service** 
3. **Traefik high HTTP 5xx error rate service**

**ExporterV1: observability/metrics/prometheus/**

1. **Traefik backend down**
2. **Traefik backend errors**
3. **Traefik high HTTP 5xx error rate backend**