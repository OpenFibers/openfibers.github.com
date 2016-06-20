---
layout: post
title: "检查iOS App是否支持IPv6-only Network"
date: 2016-06-20 14:37:10 +0800
comments: true
categories: [iOS, network, ipv6]
---

从2016年6月1号开始，苹果强制要求上架AppStore的应用支持IPv6-only network。  
本文分为两部分：通过代码检查是否兼容IPv6-only网络，以及如何搭建IPv6-only网络的测试环境。旨在帮助快速检查app是否支持IPv6-only Network。可以当做[Supporting IPv6 DNS64/NAT64 Networks](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html#//apple_ref/doc/uid/TP40010220-CH213-SW1)中**Ensuring IPv6 DNS64/NAT64 Compatibility**一节的速成版本。

### 搭建IPv6-only网络的测试环境，测试App是否能正常连接至网络

最便捷的检查方式是：找一台Mac，共享Internet到IPv6 DNS64/NAT64网络，然后让待测试设备连接到此网络，并测试app功能是否正常。  
需注意，OS X 10.11后才支持创建IPv6 DNS64/NAT64网络。如果待测试设备不是iOS或OS X或macOS设备（比如要测试Andr**d上的app是否兼容IPv6-only网络），请确保设备支持[RFC6106](https://tools.ietf.org/html/rfc6106)。

用在OS X(macOS)上创建IPv6 DNS64/NAT64网络很简单，只需要：  

1. **Mac带有两块网卡**。如果是Macbook Pro with retina display或者Macbook Air，可以买一块Thunderbolt或USB转RJ45接口的网卡来解决这个问题。  
2. 打开System Preference，按住Option点击Sharing，松开Option：
3. 勾选 **Create NAT64 Network**，把来自其他网卡的Internet共享给Wifi：
4. 将测试设备连接到此Wifi，并测试App功能是否正常。

### 检查代码是否兼容IPv6-only网络

##### 1. 尽量使用高级API

尽量使用以下框架实现网络请求：  

- **WebKit**.
- **Cocoa URL loading system**. 比如NSURLSession, NSURLRequest, 和 NSURLConnection。
- **CFNetwork**.

##### 2. 避免使用IPv4地址

- **确保在代码中不会传递点分十进制表示的IPv4地址字符串(xxx.xxx.xxx.xxx)到底层网络API**：比如确保不要传递点分十进制表示的IPv4地址字符串到*getaddrinfo*或*SCNetworkReachabilityCreateWithName*。需注意前面提到的高级API则会自动转换点分十进制表示的IPv4地址字符串到IPv6，所以在使用高级API时传递点分十进制表示的IPv4地址字符串时仍然可以正常工作。虽然如此，使用高级API时也应避免直接使用点分十进制表示的IPv4地址字符串。  
- **使用地址无关的API**：尽量传递主机名或全称域名（FQDN）给*getaddrinfo*或*getnameinfo*。


##### 3. 避免使用网络检查

- 如果有条件，自己创建网络连接，检查错误状态来代替Reachability（这成本有点大，苹果有点站着说话不腰疼）。

##### 4. 不能避免使用网络检查，则正确使用Reachability API
- 不要传递0.0.0.0到**SCNetworkReachabilityCreateWithAddress**。这只会检查网络中是否有router，并不代表真正能连接到互联网。  
- 不要传递169.254.0.0（自分配的本地地址）到**SCNetworkReachabilityCreateWithAddress**来检查Wi-Fi是否激活。如需检查是否连接到Wi-Fi，请使用**kSCNetworkReachabilityFlagsIsWWAN**。

##### 5. 正确使用Address Containers

- 使用足够大的结构来存储IPv6地址来避免溢出，比如*sockaddr_storage*。

##### 6. 检查是否使用了不兼容IPv6的API

检查是否使用过IPv4特定的API，包括但不仅限于：

```
inet_addr()
inet_aton()
inet_lnaof()
inet_makeaddr()
inet_netof()
inet_network()
inet_ntoa()
inet_ntoa_r()
bindresvport()
getipv4sourcefilter()
setipv4sourcefilter()
```

如果代码中需要检查IPv4类型，则也应一并检查IPv6类型：

IPv4 | IPv6
--------|--------
AF_INET | AF_INET6
PF_INET | PF_INET6
struct in_addr | struct in_addr6
struct sockaddr_in | struct sockaddr_in6
kDNSServiceProtocol_IPv4 | kDNSServiceProtocol_IPv6

##### 7. 连接IPv4-only的服务器时，使用getaddrinfo来保证app兼容IPv6-only网络

如果app需要连接到没有域名的IPv4-only服务器，请使用*getaddrinfo*处理IPv4地址，好处是当app处于IPv6-only网络时，*getaddrinfo*会生成一个对应的IPv6地址。

下面的代码展示了如何使用*getaddrinfo*正确处理IPv4地址。内存中有4字节存储了一个IPv4地址（{192, 0, 2, 1}），代码将其转换称了string（"192.0.2.1"），然后使用*getaddrinfo*生成了对应的IPv6地址（存储了"64:ff9b::192.0.2.1"的sockaddr_in6结构体），然后尝试连接到此IPv6地址：

```
#include <sys/socket.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <err.h>
 
    uint8_t ipv4[4] = {192, 0, 2, 1};
    struct addrinfo hints, *res, *res0;
    int error, s;
    const char *cause = NULL;
 
    char ipv4_str_buf[INET_ADDRSTRLEN] = { 0 };
    const char *ipv4_str = inet_ntop(AF_INET, &ipv4, ipv4_str_buf, sizeof(ipv4_str_buf));
 
    memset(&hints, 0, sizeof(hints));
    hints.ai_family = PF_UNSPEC;
    hints.ai_socktype = SOCK_STREAM;
    hints.ai_flags = AI_DEFAULT;
    error = getaddrinfo(ipv4_str, "http", &hints, &res0);
    if (error) {
        errx(1, "%s", gai_strerror(error));
        /*NOTREACHED*/
    }
    s = -1;
    for (res = res0; res; res = res->ai_next) {
        s = socket(res->ai_family, res->ai_socktype,
                   res->ai_protocol);
        if (s < 0) {
            cause = "socket";
            continue;
        }
 
        if (connect(s, res->ai_addr, res->ai_addrlen) < 0) {
            cause = "connect";
            close(s);
            s = -1;
            continue;
        }
 
        break;  /* okay we got one */
    }
    if (s < 0) {
        err(1, "%s", cause);
        /*NOTREACHED*/
    }
    freeaddrinfo(res0);
```

需注意，*getaddrinfo*兼容自动转换至IPv6地址的特性是iOS 9.2和OS X 10.11.2以后才支持的，但是这段代码可以保证在老的操作系统上的兼容性。  

Over