---
title:  "Purge all rabbitmq queues on all vhosts"
---

#### This sequence of commands will close all connections on a RabbitMQ Node

In troubleshooting memory issues recently I wanted a way to purge all queues, on all vhosts, without manually initiating each purge. It's possible with [rabbitmqlctl](https://www.rabbitmq.com/rabbitmqctl.8.html#Monitoring,_observability_and_health_checks). This will not work for stream type queues and is not intended to provide a solution for that.

```bash
sudo rabbitmqctl list_vhosts | \
grep -vP 'Listing|name|Timeout' | \
awk '{print $1}' | \
xargs -L 1 -I {} sh -c "sudo rabbitmqctl list_queues --vhost {} | grep -vP 'Listing|name|Timeout' | awk '{print \$1}' | xargs -L 1 -I QQ sudo rabbitmqctl purge_queue --vhost {} QQ"
```