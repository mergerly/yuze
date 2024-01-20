# README

<h1>yuze🤗</h1>

## 简介

yuze 是一款纯C实现的轻量的内网穿透工具，支持正向，反向socksv5代理隧道的搭建，支持EarthWorm (ew) 的全部数据转发方式。

目前仅支持Windows平台，更多平台很快完善中......



## 使用

    普通网络环境：
        1.  (正向代理) 可控主机有公网IP且可开启任意监听端口：
    
                  +---------+     +-------------------+  
                  |HackTools| ->> | 7878 ->  Host_One |
                  +---------+     +-------------------+
    
            a) ./yuze -m s_server -l 7878
    
            b) HackTools可使用Host_One主机的7878端口提供的socks5代理
                
        2.  (反向代理) 可控主机不存在公网IP，但是可出网，通过主动连接的方式建立反向socks代理。
        类似frp
                
                                        一台可控公网IP主机                  可控内网主机
                  +---------+     +--------------------------+    |     +---------------+
                  |HackTools| ->> | 7878 ->  Host_One -> 9999 |  防火墙  | <--  Host_Two  |
                  +---------+     +--------------------------+    |     +---------------+
    
            a) ./yuze -m yuze_listen -s 7878 -e 9999
                        // 在具有公网IP的主机上添加转接隧道，将7878端口收到的代理请求转交给反连9999端口的主机
            b) ./yuze -m r_server -r 127.0.0.1 -s 9999        
                        // 将目标主机反向连接公网转接主机
    
            c) HackTools可使用Host_One主机的7878端口提供的socks5代理
            
    对于二重网络环境：        
        1.  获得目标网络内两台主机 A、B 的权限，情况描述如下：
        
                A 主机：  存在公网 IP，且自由监听任意端口，无法访问特定资源
                B 主机：  目标网络内部主机，可访问特定资源，但无法访问公网
                A 主机可直连 B 主机
                    
                                        可控边界主机A             可访问指定资源的主机B
                  +---------+     +-----------------------+      +-----------------+
                  |HackTools| ->> | 7878 -->  Host_One --> | ->>  | 9999 -> Host_Two |
                  +---------+     +-----------------------+      +-----------------+
        
            a)  ./yuze -m s_server -l 9999
                    // 在 Host_Two 主机上利用 s_server 模式启动 9999 端口的正向 socks 代理
            b)  ./yuze -m yuze_tran -s 7878 -d 127.0.0.1 -e 9999 
                    // 在Host_One上将 7878 端口收到的 socks 代理请求转交给 Host_Two 主机。
            c)  HackTools 可通过访问 127.0.0.1:7878 来使用 127.0.0.1 主机提供的 socks5 代理。
            
        2.  获得目标网络内两台主机 A、B 的权限，情况描述如下：
        
                A 主机：  目标网络的边界主机，无公网 IP，无法访问特定资源。
                B 主机：  目标网络内部主机，可访问特定资源，却无法回连公网。
    
                A 主机可直连 B 主机
    
    				  一台可控公网IP主机                    可控内网主机A         可访问指定资源的主机B
    +---------+     +--------------------------+    |    +-----------------+      +-----------------+
    |HackTools| ->> | 7878 -> Host_One -> 8888 |  防火墙  | <--  Host_Two --> | ->> | 9999 -> Host_Three |
    +---------+     +--------------------------+    |    +-----------------+      +-----------------+
    
            a)  ./yuze -m yuze_listen -s 7878 -e 8888
                        // 在 Host_One 公网主机添加转接隧道，将 7878 收到的代理请求
                        // 转交给反连 8888 端口的主机
            b)  ./yuze -m s_server -l 9999
                        // 在 Host_Three 主机上利用 s_server 模式启动 9999 端口的正向 socks 代理
            c)  ./yuze -m yuze_slave -r 127.0.0.1 -s 8888 -d 127.0.0.1 -e 9999
                        // 在 Host_Two 上，通过工具的 yuze_slave 模式，
                        // 打通Host_One:8888 和 Host_Three:9999 之间的通讯隧道
            d)  HackTools 可通过访问 1.1.1.1:1080 来使用 2.2.2.3 主机提供的 socks5 代理





## 演示 （正向代理）

![show](./img/show.gif)






# 闲谈

yuze是我学习socket网络编程后产出的工具，它帮助我深入了解了内网渗透中常见的隧道代理，流量转发的原理。最初用go语言实现了正向、反向代理，由于体积问题，改用纯C实现。它的很多的灵感来自于逆向 EarthWorm，像前辈致敬。

如果遇见实际情况解决不了的场景，欢迎PR提供问题或思路