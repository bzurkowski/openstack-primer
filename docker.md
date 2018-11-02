# Installing Docker

Install repository setup dependencies:

```bash
$ apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

Add Docker official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Set up stable repository:

```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Update apt repository index:

```bash
$ apt-get update
```

Install the latest version of Docker:

```bash
$ apt-get install docker-ce
```

Verify installation:

```bash
$ docker run hello-world
```
