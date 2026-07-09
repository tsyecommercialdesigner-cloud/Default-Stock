Dify 常见内部端口如下，你在安全组里不要放行这些：

```
PostgreSQL:      5432
Redis:           6379
Weaviate HTTP:   8080
Weaviate gRPC:   50051
API:             5001
plugin-daemon:   5002
plugin debug:    5003
Sandbox:         8194
SSRF proxy:      3128
Nginx 容器内:    80 / 443
```

