FROM redis:6.0.4-alpine
COPY redis-7006.conf /etc/redis/
ENV TZ Asia/Shanghai
EXPOSE 6379 16379
CMD [ "redis-server", "/etc/redis/redis-7006.conf" ]

