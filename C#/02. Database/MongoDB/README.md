# MongoDB
* Run container

### Run container
in `terminal`
```sh
docker run -d --rm \
    --name mongo \
    -p 27017:27017 \
    -v mongodbdata:/data/db \
    mongo
```
