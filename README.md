# docker-registry-server
Setup docker registry server, where you can storing and distributing docker images on your local network.

#### Test Environment

- CentOS Linux release 8.1.1911 (Core)
- Docker 19.03.5

#### Tools & Images

- [OpenSSL v1.1.1d Win64 Light](https://slproweb.com/products/Win32OpenSSL.html "OpenSSL v1.1.1d Win64 Light")
- [Docker Registry 2.0](https://hub.docker.com/_/registry "Docker Registry 2.0")

#### SSL Certificate

Docker Registry requests the certificate file as *.crt*  and *.key*. My certificate is *.pfx*  file. First, using OpenSSL, I extract *.key*  and *.crt*  from my certificate with *.pfx*  extension. *(Certificate extraction was done on Windows 10 operating system.)*

##### Extract *.key *  file.

First, we extract the encrypted *.key*  file.

```bash
openssl pkcs12 -in CERTIFICATE_FILE.pfx -nocerts -out keyfile-encrypted.key
```
To unencrypt the key, do:
```bash
openssl rsa -in keyfile-encrypted.key -out keyfile.key
```
##### Extract *.crt *  file.

```bash
openssl pkcs12 -in CERTIFICATE_FILE.pfx -clcerts -nokeys -out certfile.crt
```

#### Docker Register Server

- Goto root directory. 
- Create `root/certs` directory and copy the *.crt* and *.key*  files into the directory.
- Deploy the docker registry server container with certificate files. *(If the docker registry image is not installed, it will be pull automatically.)*

```bash
docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/certfile.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/keyfile.key \
  -p 443:443 \
  registry:2
```

[Deploy your registry using a Compose file.](https://github.com/barisates/docker-registry-server/blob/master/docker-compose.yml "Deploy your registry using a Compose file.")

#### Fixing Certificate Issue

After completing the installation, you may get an error as follows during docker pull and push operations; ***Error response from daemon: Get https://registry.yourdomain.com/v2/: x509: certificate signed by unknown authority***

To fix this problem;
- Go to the `/etc/docker/certs.d` directory.
- Create a folder with the same name as your domain address.
- Copy your *.crt* file to this folder. 

This example should be like this;
`/etc/docker/certs.d/registry.yourdomain.com/certfile.crt`

##### Resources
- [Docker registry deploying.](https://docs.docker.com/registry/deploying/ "Docker registry deploying.")
- [Certificate extract.](https://nerdia.net/2018/06/16/how-to-convert-a-certificate-pfx-file-to-crt-key-using-openssl/ "Certificate extract.")