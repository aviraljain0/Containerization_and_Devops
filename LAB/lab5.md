# Experiment 5: Docker - Volumes, Environment Variables, Monitoring & Networks

---

##  Objective

To understand and implement:

- Docker Volumes for persistent storage
- Environment Variables configuration
- Docker Monitoring commands
- Docker Networks for container communication
- Multi-container real-world application setup

---

#  Part 1: Docker Volumes - Persistent Data Storage

##  Lab 1: Understanding Data Persistence

### The Problem: Container Data is Ephemeral

```bash
# Create a container that writes data
docker run -it --name test-container ubuntu /bin/bash

# Inside container:
echo "Hello World" > message.txt
cat message.txt
exit
# delete and make a new container
docker stop test-container
docker rm test-container
docker run -it --name test-container ubuntu /bin/bash
cat message.txt
```
![images for exp 5](./images/image1.jpeg)
![images for exp 5](./images/image2.jpeg)
File does not exist — Data is lost.


#  Lab 2: Volume Types

## 1. Anonymous Volumes

```bash
docker run -d -v /app/data --name web1 nginx
docker volume ls
docker inspect web1 | grep -A 5 Mounts
```
![images for exp 5](./images/image3.jpeg)
---

## 2. Named Volumes

```bash
docker volume create mydata
docker run -d -v mydata:/app/data --name web2 nginx
docker volume ls
docker volume inspect mydata
```
![images for exp 5](./images/image4.jpeg)
---

## 3. Bind Mounts (Host Directory)

```bash
mkdir ~/myapp-data
docker run -d -v ~/myapp-data:/app/data --name web3 nginx
echo "From Host" > ~/myapp-data/host-file.txt
docker exec web3 cat /app/data/host-file.txt
```
![images for exp 5](./images/image5.jpeg)

---

# Lab 3: Practical Volume Examples

## Example 1: MySQL Persistent Storage

```bash
docker run -d \
  --name mysql-db \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

docker stop mysql-db
docker rm mysql-db

docker run -d \
  --name new-mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

![images for exp 5](./images/image6.jpeg)

---

## Example 2: Nginx Configuration via Bind Mount

```bash
mkdir ~/nginx-config

echo 'server {
    listen 80;
    server_name localhost;
    location / {
        return 200 "Hello from mounted config!";
    }
}' > ~/nginx-config/nginx.conf

docker run -d \
  --name nginx-custom \
  -p 8080:80 \
  -v ~/nginx-config/nginx.conf:/etc/nginx/conf.d/default.conf \
  nginx

curl http://localhost:8080
```

![images for exp 5](./images/image7.jpeg)

---

## Lab 4: Volume Management Commands

```bash
docker volume ls
docker volume create app-volume
docker volume inspect app-volume
docker volume prune
docker volume rm volume-name
docker cp local-file.txt container-name:/path/in/volume
```
---

# Part 2: Environment Variables

## Lab 1: Using -e Flag

```bash
docker run -d \
  --name app1 \
  -e DATABASE_URL="postgres://user:pass@db:5432/mydb" \
  -e DEBUG="true" \
  -p 3000:3000 \
  my-node-app
```

---

## Using --env-file

```bash
echo "DATABASE_HOST=localhost" > .env
echo "DATABASE_PORT=5432" >> .env
echo "API_KEY=secret123" >> .env

docker run -d \
  --env-file .env \
  --name app2 \
  my-app
```

![images for exp 5](./images/image8.jpeg)

---

## Dockerfile Environment Variables

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV APP_VERSION=1.0.0
```

---

## Flask Example

```python
import os
from flask import Flask

app = Flask(__name__)

db_host = os.environ.get('DATABASE_HOST', 'localhost')
debug_mode = os.environ.get('DEBUG', 'false').lower() == 'true'
api_key = os.environ.get('API_KEY')

@app.route('/config')
def config():
    return {
        'db_host': db_host,
        'debug': debug_mode,
        'has_api_key': bool(api_key)
    }

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port, debug=debug_mode)
```


---

# 🔹 Part 3: Docker Monitoring

## docker stats

```bash
docker stats
docker stats --no-stream
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```
---

## docker top

```bash
docker top container-name
docker top container-name -ef
```
---

## docker logs

```bash
docker logs container-name
docker logs -f container-name
docker logs --tail 100 container-name
```
---

## docker inspect & events

```bash
docker inspect container-name
docker events
```
![images for exp 5](./images/image9.jpeg)
---

#  Part 4: Docker Networks

## List Networks

```bash
docker network ls
```

---

## Custom Bridge Network

```bash
docker network create my-network
docker network inspect my-network
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx
docker exec web1 curl http://web2
```

---

## Host Network

```bash
docker run -d --name host-app --network host nginx
curl http://localhost
```

![images for exp 5](./images/image10.jpeg)
---

#  Part 5: Complete Real-World Example

## Architecture

- Flask App (Port 5000)
- PostgreSQL (5432)
- Redis (6379)
- Custom Network: myapp-network

---

## Implementation

```bash
docker network create myapp-network

docker run -d \
  --name postgres \
  --network myapp-network \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_DB=mydatabase \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

docker run -d \
  --name redis \
  --network myapp-network \
  -v redis-data:/data \
  redis:7-alpine

docker run -d \
  --name flask-app \
  --network myapp-network \
  -p 5000:5000 \
  -v $(pwd)/app:/app \
  -v app-logs:/var/log/app \
  -e DATABASE_URL="postgresql://postgres:mysecretpassword@postgres:5432/mydatabase" \
  -e REDIS_URL="redis://redis:6379" \
  -e DEBUG="false" \
  -e LOG_LEVEL="INFO" \
  --env-file .env.production \
  flask-app:latest
```
![images for exp 5](./images/image11.jpeg)

![images for exp 5](./images/image12.jpeg)

![images for exp 5](./images/image13.jpeg)

![images for exp 5](./images/image14.jpeg)

![images for exp 5](./images/image15.jpeg)

![images for exp 5](./images/image16.jpeg)
---

#  Key Takeaways

- Volumes persist data beyond container lifecycle
- Environment variables configure applications dynamically
- Monitoring helps debug performance issues
- Custom networks enable container communication
- Bind mounts override container directories
- Always verify container status using `docker ps`

---
