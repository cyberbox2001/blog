# 学习进度

## 2020-8-8

- consul部分概念与操作 agent，运行可各主机节点，可以是server,也可以是node，实现把请后转发给本地的server或node服务
`
https://learn.hashicorp.com/tutorials/consul/get-started-service-discovery?in=consul/getting-started
`

- dig 命令 用于查询dns信息，consul提供自己的dns查询机制，端口8600，dig @127.0.0.1(本地ip) -p 8600 web.service.consul SRV
- socat 命令 功能强大，可以简单的建立连接，端口转发等 socat - ./1.txt ##把 1.txt 转出到的标准输出  socat -v tcp-l:8181,fork exec:"/bin/cat" ##监听8181，并输出到cat
- nc 命令 nc 127.0.0.1 8181 ##向本地 8181 输入数据，与socat配合使用
