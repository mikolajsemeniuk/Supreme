# MongoDB
* Run container
* Add driver

### Run container
in `terminal`
```sh
docker run -d --rm \
    --name mongo \
    -p 27017:27017 \
    -v mongodbdata:/data/db \
    mongo
```
### Add driver
in `terminal`
```sh
dotnet add package MongoDB.Driver
```
