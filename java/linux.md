## 静态ip

> su
>
> vi /etc/sysconfig/network-scripts/ifcfg-ens33
>
> 修改
>
> BOOTPROTO="static"
>
> ONBOOT="yes"
>
> 增加
>
> IPADDR="192.168.85.152" 
>
> NETMASK="255.255.255.0" 
>
> GATEWAY="192.168.85.2"
>
> 重启
>
> systemctl restart network

