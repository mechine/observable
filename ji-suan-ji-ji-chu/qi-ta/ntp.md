# NTP

时间是系统最基本也是最重要的部分之一，系统中的文件读写、传输等都是以系统时间为准，很多业务也要求服务器的系统时间保持一致，这就需要使用时间同步服务。目前主要有两种协议提供了时间同步服务：

NTP/SNTP协议(RFC1305): 基于udp协议(123端口)，可以估算数据包在网络上的延迟和计算机时间偏差，提供毫秒级的精度，是目前网络时间服务器使用的主流协议 TIME协议 (RFC868) : 该协议比较简单，使用tcp/udp的37端口，服务会返回从1900年1月1日午夜到现在的秒数，精度低，使用较少

NTP协议 ： ntpdate/ntp(d) TIME协议： rdate 手动同步 ： date

对时间要求不高时使用手动同步或者ntpdate+cronjob来同步时间即可 如果业务对时间非常敏感，不允许时间跃变，可以用ntpd平滑同步 在一些有防护的机房，默认丢弃所有进出的udp包，导致ntpdate/ntpd无法同步时间，这时就需要用TCP协议来同步时间

NTP 服务常见的问题： 时钟漂移 回滚 精度 网络时延抖动 安全性问题
