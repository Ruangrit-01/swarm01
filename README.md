# REFERENCE

- [https://github.com/docker/awesome-compose/tree/master/django](https://github.com/docker/awesome-compose/tree/master/django)
- [Special-Problems-in-ComNet'22-CPE-RMUTT](https://youtube.com/playlist?list=PLJz1XVERx6ACV-vTC6eG7HSMdBUR0dZId)
- [https://github.com/nantawatCharoenrat](https://github.com/nantawatCharoenrat/swarm01/blob/master/README.md)

# SWARM-DEPLOY-DJANGO-SWARM17

[ruangrit17-django.xops.ipv9.me/](https://ruangrit17-django.xops.ipv9.me/)

# WAKATIME-SWARM01-SPCN17
[WAKATIME](https://wakatime.com/@spcn17/projects/gwknykrjob?start=2023-02-27&end=2023-03-05)

# Setup-Linux
- คำสั่งที่ใช้ในการ อัพเดทและอัพเกรดแพ็คเกจ
```
sudo apt update; sudo apt upgrade -y
```
- Set timezone
```
sudo time datectl set-timezone Asia/Bangkok
```
 - คำสั่งที่ใช้ในการ Set ชื่อ Hostname
```
sudo hostnamectl set-hostname **ชื่อที่ต้องการจะตั้ง**
```
- คำสั่งที่ใช้ในการเปลี่ยน Machine-ID 
```
rm /var/lib/dbus/machine-id
echo -n > /etc/machine-id
cat /etc/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
```

# INSTALL-DOCKER
- คำสั่งที่ใช้ในการ Install Docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- คำสั่งตรวจสอบเวอร์ชั่น Docker
```
sudo docker version
```
- คำสั่งตรวจสอบการใช้งาน Docker
```
sudo docker run hello-world
```
# BUILD-IMAGE & TAG
- คำสั่งการ Build image
```
sudo docker compose "django/compose.yaml" up -d --build
```
- คำสั่งการ Tag
```
sudo docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

# PUSH IMAGE TO DOCKER HUB 
- คำสั่งเข้าสู่ระบบ Docker ใน VSCODE
```
docker login
```
- คำสั่ง Push Image To Docker Hub
```
docker push TARGET_IMAGE[:TAG]
```

# CREATE STACK DEPLOY
- สร้างไฟล์ docker-compose.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    volumes:
      - static_data:/usr/src/app/static
    ports:
      - "9088:8000"
    depends_on:
      - db
    deploy:
      replicas: 1
      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
```
- นำ docker-compose.yaml ไป Stack Deploy on local

# SWARM CLUSTER
- สร้างไฟล์ docker-compose-RevProxy.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - default
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    networks:
     - webproxy
     - default
    volumes:
      - static_data:/usr/src/app/static
    depends_on:
      - db
    deploy:
      replicas: 1
      labels:
        - traefik.docker.network=webproxy
        - traefik.enable=true
        - traefik.http.routers.${APPNAME}-https.entrypoints=websecure
        - traefik.http.routers.${APPNAME}-https.rule=Host("${APPNAME}$ URL ")
        - traefik.http.routers.${APPNAME}-https.tls.certresolver=default
        - traefik.http.services.${APPNAME}.loadbalancer.server.port=8000


      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
  webproxy:
    external: true
```

![image](https://user-images.githubusercontent.com/119097836/224471139-4c5a2e47-4281-4d6b-b74c-156a0ab10ba6.png)
