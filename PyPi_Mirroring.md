# PyPi mirroring

The goal is to provide a custom PyPi repository hosting a custom list of packages. This is useful for instance when working
on an isolated network with no internet access and you don't want  (or can't)  to host a complete PyPi mirror (to this date
~7TB of data).

## 0. Requirements

a. A machine connected to Internet with Python and Docker installed
b. A USB stick or Hard Drive
c. Docker installed on the isolated machine (which will host the PyPi mirror)

## 1. Setup

These steps require you to be on an internet-connected machine.

Write down a list of packages you want to host. For instance using pip freeze :

```
pip freeze > requirement.txt                              # to get all packages with specific versions
```

or

```
pip freeze | awk -F= '{print $1}' > requirements.txt      # to get all packages with the latest versions
```

Then install python-pypi-mirror :

```
pip install python-pypi-mirror
```

## 2. Download packages & Docker image

The next step is to get the packages from the official PyPi repository.

```
pypi-mirror download -d ./packages -r requirements.txt --binary    # --python-version 3.5 (optional) 
```

This will collect the packages wheels and metadata files and save them into the `./packages` directory.
Transfer the `./packages` directory onto your portable media.

You also need to download the `pypiserver/pypiserver:latest` image from the official Docker repository and then save it for your isolated server machine.

```
docker pull pypiserver/pypiserver:latest
docker save -o pypiserver-image.tar pypiserver/pypiserver:latest
```

This will save the pypiserver Docker image in a tarball archive. Put this archive on your USB stick or Hard Drive and pass
it to the isolated machine along with the packages directory. 

## 3. Server Setup

The next step is to prepare your host machine to serve the PyPi mirror. First, you need to have Docker installed on your
internet machine. There, run :

```
docker load -i pypiserver-image.tar
```

Then move over your packages folder into the system's `/var/pypi/packages` directory. The final step is to run the PyPi server
using docker like this :

```
docker run -v /var/pypi/packages:/data/packages -p 8080:8080 pypiserver/pypiserver
```

or

```
docker run -v /var/pypi/packages:/data/packages -p 8080:8080 -d pypiserver/pypiserver
```

to run it in the background.

## 4. Usage

```
pip install -i http://<server_ip>:8080 [packages]
```

You might want to configure pip so you don't need to type in the index all the time. Edit your `/etc/pip.conf` file and write :

```
[global]
index = http://<server_ip>:8080
index-url = http://<server_ip>:8080
trusted-host = <server_ip>
timeout = 300
```

and save. The use pip as you would on an internet machine.

## 5. Adding packages afterwards

You can simply `pip download -r requirements.txt` and directly copy the resulting files into your server's `/var/pypi/packages` directory.
