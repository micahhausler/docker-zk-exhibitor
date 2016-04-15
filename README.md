# docker-zk-exhibitor

Builds a docker image for zookeeper with exhibitor, based off of
[mbabineau/docker-zk-exhibitor](https://github.com/mbabineau/docker-zk-exhibitor).



## Build

```
docker build -t micahhausler/docker-zk-exhibitor .
```

## Run with compose


```
docker-compose up -d
docker-compose scale zk=3
docker-compose ps
```

Then connect the Exhibitor web UI on one of the docker ports mapped to `8181`

Example:

```
open http://$(docker-machine ip default):33053/
```

Give the cluster 5-7 minutes to get balance out and recognize all nodes. I've seen quite a bit of flapping
when starting nodes. Once you get 2 that are connected to each other, try setting the Settling Period to
`180000` (ms).


### Versions
* Exhibitor 1.5.6
* ZooKeeper 3.4.8

### Usage
The container expects the following environment variables to be passed in:

* `HOSTNAME` - addressable hostname for this node (Exhibitor will forward users of the UI to this address)
* `S3_BUCKET` - (optional) bucket used by Exhibitor for backups and coordination
* `S3_PREFIX` - (optional) key prefix within `S3_BUCKET` to use for this cluster
* `AWS_ACCESS_KEY_ID` - (optional) AWS access key ID with read/write permissions on `S3_BUCKET`
* `AWS_SECRET_ACCESS_KEY` - (optional) secret key for `AWS_ACCESS_KEY_ID`
* `AWS_REGION` - (optional) the AWS region of the S3 bucket (defaults to `us-west-2`)
* `ZK_PASSWORD` - (optional) the HTTP Basic Auth password for the "zk" user
* `ZK_DATA_DIR` - (optional) Zookeeper data directory
* `ZK_LOG_DIR` - (optional) Zookeeper log directory
* `AUTO_MANAGE_SETTLING_PERIOD` - (optional) The ammount in milliseconds for Exhibitor to wait before adding/removing nodes
* `HTTP_PROXY_HOST` - (optional) HTTP Proxy hostname
* `HTTP_PROXY_PORT` - (optional) HTTP Proxy port
* `HTTP_PROXY_USERNAME` - (optional) HTTP Proxy username
* `HTTP_PROXY_PASSWORD` - (optional) HTTP Proxy password

Starting the container:

    docker run -p 8181:8181 -p 2181:2181 -p 2888:2888 -p 3888:3888 \
        -e S3_BUCKET=<bucket> \
        -e S3_PREFIX=<key_prefix> \
        -e AWS_ACCESS_KEY_ID=<access_key> \
        -e AWS_SECRET_ACCESS_KEY=<secret_key> \
        -e HOSTNAME=<host> \
        micahhausler/docker-zk-exhibitor:latest

Once the container is up, confirm Exhibitor is running:

    $ curl -s localhost:8181/exhibitor/v1/cluster/status | python -m json.tool
    [
        {
            "code": 3,
            "description": "serving",
            "hostname": "<host>",
            "isLeader": true
        }
    ]
_See Exhibitor's [wiki](https://github.com/Netflix/exhibitor/wiki/REST-Introduction) for more details on its REST API._

You can also check Exhibitor's web UI at `http://<host>:8181/exhibitor/v1/ui/index.html`

Then confirm ZK is available:

    $ echo ruok | nc <host> 2181
    imok

### AWS IAM Policy
Exhibitor can also use an IAM Role attached to an instance instead of passing access or secret keys. This is an example policy that would be needed for the instance:

```
{
    "Statement": [
        {
            "Resource": [
                "arn:aws:s3:::exhibitor-bucket/*",
                "arn:aws:s3:::exhibitor-bucket"
            ],
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:DeleteObject",
                "s3:GetBucketAcl",
                "s3:GetBucketPolicy",
                "s3:GetObject",
                "s3:GetObject",
                "s3:GetObjectAcl",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:ListMultipartUploadParts",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Effect": "Allow"
        }
    ]
}
```

Starting the container:

```
docker run -p 8181:8181 -p 2181:2181 -p 2888:2888 -p 3888:3888 \
    -e S3_BUCKET=<bucket> \
    -e S3_PREFIX=<key_prefix> \
    -e HOSTNAME=<host> \
    micahhausler/docker-zk-exhibitor:latest
```

## License

MIT License, see [LICENSE](./LICENSE)
