###PHP CURL TIME_WAIT


####1.php-fpm的情况下

每个php-fpm进程下的每个请求，每次调用curl_close()后都会留下一个TIME_WAIT
我们微服务是http方式，高并发的情况下，会将TIME_WAIT打满，每次建连导致资源的浪费，而且可能会低概率的情况小出现一些问题。

kms-user 相关token解析至少会有一个，TIME_WAIT，也就是每个用户请求至少有一个，获取其他信息，连接其他服务都会产生多个WAIT。

解决：对curl扩展进行修改，使得每个进程对每个ip端口只有1个EST



###PHP REDIS TIME_WAIT

####php-fpm情况
短连接会导致多个TIME_WAIT，但是连接完成后会马上释放，

长链接会导致多个 ESTABLISHED ,每个进程每个redis一个,
php-fpm业务在复杂环境下，可能会连接多个redis，连接完成后连接不会释放，最终会每个fpm进程建立所有redis的链接

单服务器瓶颈: 200个FPM * 100 = 20000个EST  FPM增长

单redis扩展瓶颈: 每个redis200 *40台服务器=8000个EST，服务器增长

所以使用短连比较适合

####swoole情况
场景：异步服务，用redis做队列通信，每个worker不断从redis中获取pop数据消费
短连接会导致 超大量TIME_WAIT 很快打满

长连接，单个服务连接的redis的数量很少，通常只有一个，所以产生 worker数量*redis数量=ESTABLISHED

所以使用长连比较适合


如果继续扩展机器：

1.共享内存连接池？？
2.业务拆分、下沉
3.业务服务化
4.换语言重构