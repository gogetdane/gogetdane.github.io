---
title:  "Close all rabbitmq connections"
---

#### This sequence of commands will close all connections on a RabbitMQ Node

In troubleshooting memory issues recently I wanted a way to explicitly close all connections on a RMQ node.

```bash
sudo rabbitmqctl list_connections --node YOUR_NODE_NAME \
pid port state user vhost recv_cnt send_cnt send_pend name | \
grep -vP 'Listing|pid' | \
awk '{print $1}' | \
xargs -L 1 -I {} sudo rabbitmqctl close_connection '{}' "Closed by @gogetdane"
```

There is also a built in command for this, which is available in the [RMQ docs](https://www.rabbitmq.com/rabbitmqctl.8.html#Connection_Operations).