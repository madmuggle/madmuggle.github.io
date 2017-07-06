# 不用把OPTIONS写在Access-Control-Allow-Methods里
2017/04/07 16:56:00


之前在回复跨域请求的时候，总是习惯直接把Access-Control-Allow-Methods这个HTTP头设置成：'GET,POST,OPTIONS'。今天发现这样不仅不必要，逻辑上讲还有点搞笑。

从协议角度来讲，Access-Control-Allow-Methods是对Access-Control-Request-Method的直接回应，而后者只有OPTIONS请求会发。

所以Access-Control-Allow-Method就是和OPTIONS请求配套使用的。

这引出两个结论：

1. 并不是所有的跨域请求都需要回复Access-Control-Allow-Methods，只有OPTIONS需要你回复这个。

2. 不同于其他跨域请求，OPTIONS永远都不会被浏览器阻止。

所以在服务端，更合适的做法是检测一下请求的方法，然后根据需要决定是否回复Access-Control-Allow-Method这个头。

