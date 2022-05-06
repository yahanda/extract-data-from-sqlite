# extract-data-from-sqlite
IoT Edge module for retrieving data from SQLite

## Build

```
sudo docker build -t extractdatafromsqlite -f Dockerfile.arm32v7 .
sudo docker tag extractdatafromsqlite <your-container-registry>/extractdatafromsqlite:1.0
sudo docker login -u <username> -p <password> <your-container-registry>
sudo docker push <your-container-registry>/extractdatafromsqlite:1.0
```

If _cgroup mountpoint does not exist_ error occurs when running docker build command, you may use the following workaround.
```
# https://bugs.mageia.org/show_bug.cgi?id=27251
sudo mkdir /sys/fs/cgroup/systemd
sudo mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd
```

## Deploy

### Environment Variables
|Name|Value|
|-|-|
|DB_NAME| name of the database |
|SQL_STATEMENT| statement used to select data from a database |
|INTERVAL_SEC| interval in seconds for querying data and sending message to IoT Hub

### Container Create Options
Bind the location of the SQLite database file (normally it has `.db`, `.sqlite`, or `.sqlite3` file extension). If it is located under `/sqlite` on the host, set the container create options as follows.
```
{
    "HostConfig": {
        "Binds": [
            "/sqlite:/app/db"
        ]
    }
}
```
