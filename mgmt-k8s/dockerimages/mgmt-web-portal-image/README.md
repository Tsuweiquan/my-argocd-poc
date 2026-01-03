# How to push image?
```bash
# Image name: map-<container name>:v<version>
docker login
docker build . -t tsuweiquan/map-mgmt-web-portal:v1.0.0
docker push tsuweiquan/map-mgmt-web-portal:v1.0.0
```