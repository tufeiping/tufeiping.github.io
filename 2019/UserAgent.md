## 常见浏览器User Agent及应用

> 原创 涂飞平  `2018-12-18`

最近在做App二维码下载的功能，需要在后端判断扫描来自PC，Android手机还是苹果手机，所以将常见的浏览器User Agent记录下来。

~~~txt
Firefox linux: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:64.0) Gecko/20100101 Firefox/64.0
~~~

~~~txt
Chrome windows: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36
~~~

~~~txt
Chrome linux: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/71.0.3578.80 Chrome/71.0.3578.80 Safari/537.36
~~~

~~~txt
IE8: Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727)
~~~

~~~txt
Android Vivo x21: Mozilla/5.0 (Linux; Android 8.1.0; vivo X21UD A Build/OPM1.171019.011; wv) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.84 Mobile Safari/537.36 VivoBrowser/5.6.3.2
~~~

~~~txt
Android Wexin: Mozilla/5.0 (Linux; Android 8.1.0; vivo X21UD A Build/OPM1.171019.011; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/66.0.3359.126 MQQBrowser/6.2 TBS/044425 Mobile Safari/537.36 MMWEBID/3696 MicroMessenger/6.7.3.1360(0x2607033C) NetType/WIFI Language/zh_CN Process/tools
~~~

~~~txt
iOS: Mozilla/5.0 (iPhone; CPU iPhone OS 12_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0 Mobile/15E148 Safari/604.1
~~~

~~~txt
iOS: Mozilla/5.0 (iPhone; CPU iPhone OS 11_4 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/11.0 Mobile/15E148 Safari/604.1
~~~

~~~txt
iOS Weixin: Mozilla/5.0 (iPhone; CPU iPhone OS 7_0_2 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) CriOS/30.0.1599.12 Mobile/11A501 Safari/8536.25 MicroMessenger/6.1.0
~~~

之所以把微信的内嵌浏览器User-Agent单列出来，是因为使用微信扫码会中止跳转，需要使用“在浏览器打开”才能完成接下来的动作。

<p style="text-align: center;"><img src="https://raw.githubusercontent.com/tufeiping/tufeiping.github.io/master/assets/useragent.png" alt="useragent.png"></p>

至于判断本身逻辑就比较简单了，如下：
~~~javascript
const weixinTip = '';
app.get('/getappaddr', (req, res) => {
  let userAgent = req.headers['user-agent'];
  if (/MQQBrowser/g.test(userAgent) || /MicroMessenger/g.test(userAgent)) { // open page with weixin
    res.send(weixinTip);
  } else if (/Android/g.test(userAgent)) { // Android browser
    res.redirect('http://xxxxx/xxxx.apk');
  } else if (/Mac OS/g.test(userAgent)) { // iOS browser
    res.redirect('https://itunes.apple.com/cn/app/apple-store/idxxxxxx?mt=8');
  } else { // others (eg. pc) not support
    res.send('Request not support');
  }
});
~~~