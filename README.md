    Copyright 2019 Kedu S.C.C.L.

    Email-to-ticket is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    Foobar is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with Foobar.  If not, see <http://www.gnu.org/licenses/>.

    info@kedu.coop

# For the impatients

## With VPN username

1. Run the container

```
docker run --name snx-vpn \
  --cap-add=ALL \
  -v /lib/modules:/lib/modules \
  -e SNX_SERVER=vpn_server_ip_address \
  -e SNX_USER=user \
  -e SNX_PASSWORD=secret \
  -ti \
  -d kedu/snx-checkpoint-vpn
```

2. Get private IP address of docker container

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' snx-vpn
172.17.0.2
```

3. Add a route using previous step IP address as gateway

```
sudo route add -net 10.20.30.0 gw 172.17.0.2 netmask 255.255.255.0
```

4. Try to reach the server behind SNX VPN (in this example through SSH)

```
ssh 10.20.30.40
```

## With username and certificate

1. Run the container

```
docker run --name snx-vpn \
  --cap-add=ALL \
  -v /lib/modules:/lib/modules \
  -e SNX_SERVER=vpn_server_ip_address \
  -e SNX_USER=user \
  -e SNX_PASSWORD=secret \
  -e SNX_CERTIFICATE="my_snx_vpn_certificate.p12" \
  -v /path/to/my_snx_vpn_certificate.p12:/my_snx_vpn_certificate.p12 \
  -ti \
  -d kedu/snx-checkpoint-vpn
```

Notes:

* "SNX_CERTIFICATE" should match the container destination filename
* The container destination directory must be "/"

2. Get private IP address of docker container

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' snx-vpn
172.17.0.2
```

3. Add a route using previous step IP address as gateway

```
sudo route add -net 10.20.30.0 gw 172.17.0.2 netmask 255.255.255.0
```

4. Try to reach the server behind SNX VPN (in this example through SSH)

```
ssh 10.20.30.40
```

## Without username and with certificate

1. Run the container

```
docker run --name snx-vpn \
  --cap-add=ALL \
  -v /lib/modules:/lib/modules \
  -e SNX_SERVER=vpn_server_ip_address \
  -e SNX_PASSWORD=secret \
  -e SNX_CERTIFICATE="my_snx_vpn_certificate.p12" \
  -v /path/to/my_snx_vpn_certificate.p12:/my_snx_vpn_certificate.p12 \
  -ti \
  -d kedu/snx-checkpoint-vpn
```

Notes:

* "SNX_CERTIFICATE" should match the container destination filename
* The container destination directory must be "/"

2. Get private IP address of docker container

```
docker inspect --format '{{ .NetworkSettings.IPAddress }}' snx-vpn
172.17.0.2
```

3. Add a route using previous step IP address as gateway

```
sudo route add -net 10.20.30.0 gw 172.17.0.2 netmask 255.255.255.0
```

4. Try to reach the server behind SNX VPN (in this example through SSH)

```
ssh 10.20.30.40
```

# Troubleshooting

This image has just been tested without username and with certificate, and with snx build 800007075 obtained from:

https://www.fc.up.pt/ci/servicos/acesso/vpn/software/CheckPointVPN_SNX_Linux_800007075.sh

If you can't connect to your Checkpoint VPN server try using other SNX builds, for instance:

https://supportcenter.checkpoint.com/supportcenter/portal/user/anon/page/default.psml/media-type/html?action=portlets.DCFileAction&eventSubmit_doGetdcdetails=&fileid=8993

If the container started up you could quickly test the new SNX build as follows:

1. Copy the SNX build from the docker host to the docker container

```
docker cp snx_install.sh snx-vpn:/
```

2. Connect to the docker container

```
docker exec -ti snx-vpn bash
```

3. Get process ID of the currently running SNX client (if any):

```
ps ax
```

Expected output similar to:

```
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss     0:00 /bin/bash /root/snx.sh
   29 ?        Ss     0:00 snx -s ip_vpn_server -c /vpn_certificate_filename.p12
   32 pts/0    S+     0:00 /bin/bash
   37 pts/1    Ss     0:00 bash
   47 pts/1    R+     0:00 ps ax
```

4. Kill the process (in this example 29):

```
kill 29
```

5. Adjust permissions of the SNX build

```
chmod a+rx snx_install.sh
```

6. Execute the installation file:

```
chmod a+rx snx_install.sh
```

Expected output:

```
        Installation successfull
```

7. Check installation:


```
ldd /usr/bin/snx
```

Expected output similar to:

```
	linux-gate.so.1 (0xf7f3c000)
	libX11.so.6 => /usr/lib/i386-linux-gnu/libX11.so.6 (0xf7dea000)
	libpthread.so.0 => /lib/i386-linux-gnu/libpthread.so.0 (0xf7dcb000)
	libresolv.so.2 => /lib/i386-linux-gnu/libresolv.so.2 (0xf7db3000)
	libdl.so.2 => /lib/i386-linux-gnu/libdl.so.2 (0xf7dae000)
	libpam.so.0 => /lib/i386-linux-gnu/libpam.so.0 (0xf7d9e000)
	libnsl.so.1 => /lib/i386-linux-gnu/libnsl.so.1 (0xf7d83000)
	libstdc++.so.5 => /usr/lib/i386-linux-gnu/libstdc++.so.5 (0xf7cc4000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7ae8000)
	libxcb.so.1 => /usr/lib/i386-linux-gnu/libxcb.so.1 (0xf7abc000)
	/lib/ld-linux.so.2 (0xf7f3e000)
	libaudit.so.1 => /lib/i386-linux-gnu/libaudit.so.1 (0xf7a92000)
	libm.so.6 => /lib/i386-linux-gnu/libm.so.6 (0xf798e000)
	libgcc_s.so.1 => /lib/i386-linux-gnu/libgcc_s.so.1 (0xf7970000)
	libXau.so.6 => /usr/lib/i386-linux-gnu/libXau.so.6 (0xf796c000)
	libXdmcp.so.6 => /usr/lib/i386-linux-gnu/libXdmcp.so.6 (0xf7965000)
	libcap-ng.so.0 => /lib/i386-linux-gnu/libcap-ng.so.0 (0xf795f000)
	libbsd.so.0 => /lib/i386-linux-gnu/libbsd.so.0 (0xf7944000)
	librt.so.1 => /lib/i386-linux-gnu/librt.so.1 (0xf793a000)
```

8. Manually try to connect:

```
snx -s ip_vpn_server -c /vpn_certificate_filename.p12
```

Expected output similar to:

```
Please enter the certificate's password:
```

9. Type the password and press "Enter"

Expected output similar to:

```
SNX - connected.

Session parameters:
===================
Office Mode IP      : 192.168.90.82
DNS Server          : 10.20.30.41
Secondary DNS Server: 10.20.30.42
Timeout             : 6hours 
```

# Make the connection easier

Once you checked that the SNX client works, you could create a script to make the whole process easier:

1. Create the script


```
sudo vim /usr/local/bin/snx-vpn.sh
```

With below content, adjusting "SNX_DOCKER_NAME" and routes to match your needs:

```
#! /bin/bash
SNX_DOCKER_NAME="snx-vpn"
SNX_DOCKER_IP="$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' $SNX_DOCKER_NAME)"
if [ -z "$SNX_DOCKER_IP" ]; then
	echo >&2 'error: missing SNX_DOCKER_IP, is $SNX_DOCKER_NAME docker running?'
	exit 1;
fi
#sudo route add -net a.b.c.0 netmask 255.255.255.0 gw $SNX_DOCKER_IP
#sudo route add -net x.y.z.0 netmask 255.255.255.0 gw $SNX_DOCKER_IP
```

2. Make it executable

```
chmod +x /usr/local/bin/snx-vpn.sh
```

3. Test it:

```
docker start snx-vpn
/usr/local/bin/snx-vpn.sh
```

# Credits

This image is inspired in the excellent work of below people:

https://github.com/iwanttobefreak/docker-snx-vpn

https://github.com/mnasiadka/docker-snx-dante

https://unix.stackexchange.com/a/453727

https://gitlab.com/jamgo/docker-juniper-vpn