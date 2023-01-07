# WEB

## Classic Childhood Game

```js
function mota() {
  var a = ['\x59\x55\x64\x6b\x61\x47\x4a\x58\x56\x6a\x64\x61\x62\x46\x5a\x31\x59\x6d\x35\x73\x53\x31\x6c\x59\x57\x6d\x68\x6a\x4d\x6b\x35\x35\x59\x56\x68\x43\x4d\x45\x70\x72\x57\x6a\x46\x69\x62\x54\x55\x31\x56\x46\x52\x43\x4d\x46\x6c\x56\x59\x7a\x42\x69\x56\x31\x59\x35'];
  (function (b, e) {
    var f = function (g) {
      while (--g) {
        b['push'](b['shift']());
      }
    };
    f(++e);
  }(a, 0x198));
  var b = function (c, d) {
    c = c - 0x0;
    var e = a[c];
    if (b['CFrzVf'] === undefined) {
      (function () {
        var g;
        try {
          var i = Function('return\x20(function()\x20' + '{}.constructor(\x22return\x20this\x22)(\x20)' + ');');
          g = i();
        } catch (j) {
          g = window;
        }
        var h = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=';
        g['atob'] || (g['atob'] = function (k) {
          var l = String(k)['replace'](/=+$/, '');
          var m = '';
          for (var n = 0x0, o, p, q = 0x0; p = l['charAt'](q++); ~p && (o = n % 0x4 ? o * 0x40 + p : p, n++ % 0x4) ? m += String['fromCharCode'](0xff & o >> (-0x2 * n & 0x6)) : 0x0) {
            p = h['indexOf'](p);
          }
          return m;
        });
      }());
      b['fqlkGn'] = function (g) {
        var h = atob(g);
        var j = [];
        for (var k = 0x0, l = h['length']; k < l; k++) {
          j += '%' + ('00' + h['charCodeAt'](k)['toString'](0x10))['slice'](-0x2);
        }
        return decodeURIComponent(j);
      };
      b['iBPtNo'] = {};
      b['CFrzVf'] = !![];
    }
    var f = b['iBPtNo'][c];
    if (f === undefined) {
      e = b['fqlkGn'](e);
      b['iBPtNo'][c] = e;
    } else {
      e = f;
    }
    return e;
  };
  alert(atob(b('\x30\x78\x30')));
}
```

在js源码里搜**alert**，发现此函数，在控制台执行一下得到flag

> hgame{fUnnyJavascript&FunnyM0taG4me}

## Become A Member







## Guess Who I Am

各种符号，表情，汉字和html编码😵

最终写了一个烂烂的代码

```python
import requests
import html

member = [
    {
        "id": "ba1van4",
        "intro": "21级 / 不会Re / 不会美工 / 活在梦里 / 喜欢做不会的事情 / ◼◻粉",
    },
]

url = 'http://week-1.hgame.lwsec.cn:31314'
session = requests.session()
r = session.get(url = url)
print(r.cookies)
while 1:
    r = session.get(url = url + '/api/getQuestion')
    a = r.text.find('message')
    question = r.text[a + 10 : -2]
    if r'\u0026lt;/p\u0026gt;' in question:
        question = '19级 / &lt;/p&gt;&lt;p&gt;Web'
    if r'\u003cdel\u003e' in question:
        question = '什么都不会 / 咸鱼研究生 / <del>安恒</del>、<del>长亭</del> / SJTU'
    if r'\u0026' in question:
        question = question.replace(r'\u0026', '&')
    print('[question]' + question)
    for i in member:
        if question == i['intro']:
            print('[answer]' + i['id'])
            data = {
                'id' : i['id']
            }
            r = session.post(url = url + '/api/verifyAnswer', data = data)
            print(r.text)
            break
    r = session.get(url = url + '/api/getScore')
    print(r.text)
    print('-----------------')
```

**member**是hint给出的，就省略了（开始还没看见，去官网抓了好久😵

> hgame{Guess_who_i_am^Happy_Crawler}

## Show Me Your Beauty

黑名单，大写绕过

![](https://raw.githubusercontent.com/xlccccc/Image/master/hexoblog/202301061239921.png)

> hgame{Unsave_F1L5_SYS7em_UPL0ad!}