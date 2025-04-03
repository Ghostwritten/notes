
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fd98c84baecc426fae47813fdf59759f.jpeg)





## docker 安装
```bash
$ docker run -it --rm -p 8006:8006 --device=/dev/kvm --cap-add NET_ADMIN --stop-timeout 120 dockurr/windows
❯ Starting Windows for Docker v3.13...
❯ For support visit https://github.com/dockur/windows
❯ CPU: 11th Gen Intel Core TM i5 1135G7 | RAM: 28/31 GB | DISK: 309 GB (xfs) | HOST: 4.18.0-553.16.1.el8_10.x86_64...

❯ Requesting Windows 11 from Microsoft server...
❯ Downloading Windows 11...
/storage/tmp/win11x64.iso            100%[======================================================================>]   6.34G  34.4MB/s    in 3m 8s   
❯ Extracting Windows 11 image...
❯ Adding drivers to image...
❯ Adding win11x64.xml for automatic installation...
❯ Building Windows 11 image...



BdsDxe: failed to load Boot0002 "UEFI QEMU QEMU HARDDISK " from PciRoot(0x0)/Pci(0xA,0x0)/Scsi(0x0,0x0): Not Found
BdsDxe: loading Boot0001 "UEFI QEMU QEMU CD-ROM " from PciRoot(0x0)/Pci(0x5,0x0)/Scsi(0x0,0x0)
BdsDxe: starting Boot0001 "UEFI QEMU QEMU CD-ROM " from PciRoot(0x0)/Pci(0x5,0x0)/Scsi(0x0,0x0)
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4e9f94a21b5144a3880d8fcc7cb84e27.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/99c1eac72b3042d4b1c96061c297cb91.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b4ebf9382b914b6d945b78ce655759b9.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4ddfddfbaba244c28becde0be9ae7cae.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c13fca554eb84a93bf92e6cc074a7022.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d2f41c6dee0640d5a27e898124c5caa7.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe69f7afdbc445aaaeadd950ec154b22.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c9cc750c066240df9c43304f2806218a.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9271f060a78146d29a55efa198081466.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e536dd7ef17f45c1aab530de6b7e2940.png)

## docker-compose 安装

```bash
$ cat docker-compose.yml
services:
  windows:
    image: dockurr/windows
    container_name: windows
    environment:
      VERSION: "win11"
      REGION: "en-US"
      KEYBOARD: "en-US"
      HTTP_PROXY: "http://192.168.21.95:7890"
      HTTPS_PROXY: "http://192.168.21.95:7890"
    devices:
      - /dev/kvm
    cap_add:
      - NET_ADMIN
    ports:
      - 8006:8006
      - 3389:3389/tcp
      - 3389:3389/udp
    stop_grace_period: 2m

networks:
  host:
    name: host
    external: true
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d030dd5d88c74cf8842aafced552c071.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2f07c692749f433eadb6762d264a2140.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a2fce14420ab46e1bee8c3cc032617ef.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/404cfbacf2ab4d3980ce43963ad0a42e.png)

