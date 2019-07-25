查看某个端口的访问连接数：netstat -nat|grep -i "9091"|wc -l
查看端口启用情况：netstat -lnp
netstat -nat|grep 9091|grep -v "TIME_WAIT"|wc -l
抓包：tcpdump -i any -p -s 0 -w  ~/test777.pcap &

服务重新挂载:mount -o remount,rw /
               maxIdleTime="20000"


chown -R  username:username  目录/文件夹
