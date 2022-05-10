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
|INTERVAL_SEC| interval in seconds for querying data and sending message to IoT Hub |

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

## Example
Download [SQLite Sample Database](https://www.sqlitetutorial.net/sqlite-sample-database/) and extract it to `/sqlite` directory. The name of the file is `chinook.db`.
```
sudo mkdir -p -m 777 /sqlite
cd /sqlite
wget https://www.sqlitetutorial.net/wp-content/uploads/2018/03/chinook.zip
unzip chinook.zip
ls chinook.db
```

Deploy the IoT Edge module with the following setting.

### Environment Variables
|Name|Value|
|-|-|
|DB_NAME| chinook.db |
|SQL_STATEMENT| select * from employees limit 5 |
|INTERVAL_SEC| 10 |

### Container Create Options
```
{
    "HostConfig": {
        "Binds": [
            "/sqlite:/app/db"
        ]
    }
}
```

You will receive IoT Hub messages like the following every 10 seconds.
```
{
  "body": [
    {
      "EmployeeId": 1,
      "LastName": "Adams",
      "FirstName": "Andrew",
      "Title": "General Manager",
      "ReportsTo": null,
      "BirthDate": "1962-02-18 00:00:00",
      "HireDate": "2002-08-14 00:00:00",
      "Address": "11120 Jasper Ave NW",
      "City": "Edmonton",
      "State": "AB",
      "Country": "Canada",
      "PostalCode": "T5K 2N1",
      "Phone": "+1 (780) 428-9482",
      "Fax": "+1 (780) 428-3457",
      "Email": "andrew@chinookcorp.com"
    },
    {
      "EmployeeId": 2,
      "LastName": "Edwards",
      "FirstName": "Nancy",
      "Title": "Sales Manager",
      "ReportsTo": 1,
      "BirthDate": "1958-12-08 00:00:00",
      "HireDate": "2002-05-01 00:00:00",
      "Address": "825 8 Ave SW",
      "City": "Calgary",
      "State": "AB",
      "Country": "Canada",
      "PostalCode": "T2P 2T3",
      "Phone": "+1 (403) 262-3443",
      "Fax": "+1 (403) 262-3322",
      "Email": "nancy@chinookcorp.com"
    },
    {
      "EmployeeId": 3,
      "LastName": "Peacock",
      "FirstName": "Jane",
      "Title": "Sales Support Agent",
      "ReportsTo": 2,
      "BirthDate": "1973-08-29 00:00:00",
      "HireDate": "2002-04-01 00:00:00",
      "Address": "1111 6 Ave SW",
      "City": "Calgary",
      "State": "AB",
      "Country": "Canada",
      "PostalCode": "T2P 5M5",
      "Phone": "+1 (403) 262-3443",
      "Fax": "+1 (403) 262-6712",
      "Email": "jane@chinookcorp.com"
    },
    {
      "EmployeeId": 4,
      "LastName": "Park",
      "FirstName": "Margaret",
      "Title": "Sales Support Agent",
      "ReportsTo": 2,
      "BirthDate": "1947-09-19 00:00:00",
      "HireDate": "2003-05-03 00:00:00",
      "Address": "683 10 Street SW",
      "City": "Calgary",
      "State": "AB",
      "Country": "Canada",
      "PostalCode": "T2P 5G3",
      "Phone": "+1 (403) 263-4423",
      "Fax": "+1 (403) 263-4289",
      "Email": "margaret@chinookcorp.com"
    },
    {
      "EmployeeId": 5,
      "LastName": "Johnson",
      "FirstName": "Steve",
      "Title": "Sales Support Agent",
      "ReportsTo": 2,
      "BirthDate": "1965-03-03 00:00:00",
      "HireDate": "2003-10-17 00:00:00",
      "Address": "7727B 41 Ave",
      "City": "Calgary",
      "State": "AB",
      "Country": "Canada",
      "PostalCode": "T3B 1Y7",
      "Phone": "1 (780) 836-9987",
      "Fax": "1 (780) 836-9543",
      "Email": "steve@chinookcorp.com"
    }
  ],
  "enqueuedTime": "Tue May 10 2022 19:01:42 GMT+0900 (Japan Standard Time)"
}
```