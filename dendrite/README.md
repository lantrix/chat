# Dendrite Install

## Install server

 * Build dendrite https://github.com/matrix-org/dendrite#get-started

```
git clone https://github.com/matrix-org/dendrite
cd dendrite
./build.sh
```

 * Copy binaries and set binary permissions:

```
for i in `ls bin/`; do echo $i ; doas cp bin/$i /usr/local/bin/ && doas chown matrix:wheel /usr/local/bin/$i ;done
```

## Dendrite Database

Outside the scope of this create a PostgreSQL DB for dendrite, you will need the connection string for the config:

```
postgres://dendrite:password@dendritedb.local/dendrite
```

## Dendrite Config

 * Copy [`dendrite.yaml`](./dendrite-example.yaml) to `~matrix/dentrite.yml`
   - Configure the `server_name` in the config
   - Set all the postgres connection strings `connection_string`

Generate a Matrix signing key for federation (required):

```shell
$ doas su - matrix -c "/usr/local/bin/generate-keys --private-key /home/dendrite/matrix_key.pem"

Created private key file: /home/dendrite/matrix_key.pem
``` 
## Dendrite Service

Copy [`dendrite-rc.conf`](./dendrite-rc.conf) to `/etc/rc.d/dendrite`

Append [`dendrite-rc.conf.local`](./dendrite-rc.conf.local) contents to `/etc/rc.conf.local`

Enable the service:

```shell
chmod 555 /etc/rc.d/dendrite
rcctl enable dendrite
rcctl start dendrite
```

Test the dendrite monolith is up:

```shell
$ tail /var/log/dendrite/Monolith.log

time="2021-02-01T03:13:24.497656445Z" level=info msg="Dendrite version 0.3.8+369d3939" func=github.com/matrix-org/dendrite/setup.NewBaseDendrite file="github.com/matrix-org/dendrite/setup/base.go:110"
time="2021-02-01T03:13:25.484905959Z" level=info msg="Starting external Monolith listener on :8008" func="github.com/matrix-org/dendrite/setup.(*BaseDendrite).SetupAndServeHTTP.func2" file="github.com/matrix-org/dendrite/setup/base.go:395"
```

Test the federation API responds

```
$ curl https://server.chat/_matrix/federation/v1/version

{"server":{"version":"0.3.8+369d3939","name":"Dendrite"}}
```