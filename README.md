# docker-registry-server
Deploy a Docker Trusted Registry (DTR) server, where you can storing and distributing docker images on your local network.

#### Test Environment

- CentOS Linux release 8.1.1911 (Core)
- Docker 19.03.5

#### Tools & Images

- [OpenSSL v1.1.1d Win64 Light](https://slproweb.com/products/Win32OpenSSL.html "OpenSSL v1.1.1d Win64 Light")
- [Docker Trusted Registry 2.0](https://hub.docker.com/_/registry "Docker Trusted Registry 2.0")

#### SSL Certificate

Docker Trusted Registry (DTR) requests the certificate file as *.crt*  and *.key*. My certificate is *.pfx*  file. First, using OpenSSL, I extract *.key*  and *.crt*  from my certificate with *.pfx*  extension. *(Certificate extraction was done on Windows 10 operating system.)*

##### Extract *.key*  file.

First, we extract the encrypted *.key*  file.

```bash
openssl pkcs12 -in CERTIFICATE_FILE.pfx -nocerts -out keyfile-encrypted.key
```
To unencrypt the key, do:
```bash
openssl rsa -in keyfile-encrypted.key -out keyfile.key
```
##### Extract *.crt*  file.

```bash
openssl pkcs12 -in CERTIFICATE_FILE.pfx -clcerts -nokeys -out certfile.crt
```

#### Docker Trusted Registry (DTR) Server

- Goto root directory.
- Create `root/certs` directory and copy the *.crt* and *.key*  files into the directory.
- Create `root/registry` directory and copy the `registry/config.yml` files into the directory.

> We configure our DTR server to accept CORS for Docker Registry UI.
```yaml
    Access-Control-Allow-Origin: ['*']
    Access-Control-Allow-Methods: ['HEAD', 'GET', 'OPTIONS', 'DELETE']
    Access-Control-Expose-Headers: ['Docker-Content-Digest']
```

- Deploy the DTR server container with certificate files and new configuration. *(If the docker registry image is not installed, it will be pull automatically.)*

```bash
docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -v "$(pwd)"/registry/config.yml:/etc/docker/registry/config.yml \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/certfile.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/keyfile.key \
  -p 443:443 \
  registry:2
```

#### Fixing Certificate Issue

After completing the installation, you may get an error as follows during docker pull and push operations; ***Error response from daemon: Get https://registry.yourdomain.com/v2/: x509: certificate signed by unknown authority***

To fix this problem;
- Go to the `/etc/docker/certs.d` directory.
- Create a folder with the same name as your domain address.
- Copy your *.crt* file to this folder.

This example should be like this;
`/etc/docker/certs.d/registry.yourdomain.com/certfile.crt`


#### Docker Trusted Registry (DTR) User Interface

We use [Docker Registry UI](https://github.com/Joxit/docker-registry-ui "Docker Registry UI") to manage our images on our DTR server through a user interface.

```bash
docker run -d \
  --restart=always \
  --name registry-ui \
  -p 80:80 \
  -e URL=https://registry.yourdomain.com \
  -e DELETE_IMAGES=true \
  joxit/docker-registry-ui:static
```

##### Resources
- [How to convert a certificate PFX file to CRT/KEY using openssl](https://nerdia.net/2018/06/16/how-to-convert-a-certificate-pfx-file-to-crt-key-using-openssl/ "How to convert a certificate PFX file to CRT/KEY using openssl")
- [Deploy a registry server](https://docs.docker.com/registry/deploying/ " Deploy a Registry Server")
- [Docker Registry UI](https://github.com/Joxit/docker-registry-ui "Docker Registry UI")