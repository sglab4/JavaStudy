浏览器请求**解析 DNS 服务器**，把域名 www.baidu.com 转换成 web 服务器的 **IP 地址**

1. 系统首先会查找本地的DNS缓存和hosts文件信息
2. 如果没有找到，那么，系统将会把浏览器的解析请求发送给本地主机所指定的DNS服务器，称为LDNS
3. LDNS 服务器会从DNS系统的根（.）开始请求对域名 www.baidu.com 的解析。根 DNS 服务器全球只有 13 台，根域名服务器是没有域名 www.baidu.com 解析记录的。但是它会有域名 www.baidu.com 所对应的顶级域 .com 的解析记录，因此直接把顶级域 .com 所对应的 DNS 地址返回给 LDNS 服务器
4. LDNS服务器获取到顶级域 .com 对应的 DNS 服务器地址后，就会去 .com 服务器请求对 www.baidu.com 域名的解析。在顶级域名服务器也不会有 www.baidu.com 的解析记录的。但是它有 www.baidu.com 的父级域名，即 baidu.com。因此顶级域名 .com 服务器又会把 baidu.com 所对应的 DNS 服务器的 IP 地址返回给 LDNS。
5. LDNS 服务器收到 baidu.com 所对应的 IP 地址后，就会去 baidu.com 域名服务器请求对 www.baidu.com 的域名解析。baidu.com 域名对应的 DNS 服务器是该域名的授权 DNS 服务器。这个 DNS 服务器就是企业购买域名时用于管理解析的服务器。
6. baidu.com 域名 DNS 服务器会把 www.baidu.com 域名所对应的 IP 地址给解析出来，然后发给 LDNS。
7. LDNS 把解析出来的结果，www.baudu.com 所对应的 IP 地址发送给客户端的浏览器。并且 LDNS 也会将其域名和对应的地址缓存到 cache 中。
8. 客户端浏览器收到后，也会将其域名以及对应的 IP 地址缓存的到 DNS 缓存和 hosts 文件中。