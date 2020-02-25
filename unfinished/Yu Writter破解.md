# Yu Writter 破解

# 绕过app文件校验

文件校验在 `app/config/security.json` 指定。

找到 mainprocess.js

搜索 『security』关键字找到下面 class

```js
class  l {
constructor(e) {
this.settingService  =  e
}
getSalt() {
return  this.settingService.defaults.library.component.virtual.name
}
getSignature() {
return  this.settingService.defaults.security.sign
}

check(e) {
let  t  =  s.createVerify("RSA-SHA256"),
o  =  this.settingService.runtime.paths.app,
a  = () => {
let  i  =  this.getSignature(),
// 注释前
// r = t.verify(n, i, "base64");
// 注释后
r  =  true;
e(null, r)
},
l  =  s  => {
if (0  ===  s.length) return  void  a();
let  n  =  s.pop(),
d  =  i.join(o, n);
r.readFile(d, (i, r) => {
i  ?  e(i) : (t.update(r), l(s))
})
},
d  =  this.getSalt();
t.update(d), l(["package.json", "main.js", "src/script/mainprocess.js", "src/script/mainwindowpage.js", "src/script/renderwindowpage.js", "src/script/browserprocess.js", "src/config/base.json", "src/config/library.json", "src/i18n/message-en.yaml", "src/i18n/message-zh.yaml", "src/i18n/message-zh-HK.yaml", "src/i18n/message-zh-TW.yaml", "src/view/main.html", "src/view/render.html", "src/styles/main.css", "src/images/alipay.jpg", "src/images/wechat-pay.jpg", "src/builtinlibraries/Manual/en/Welcome.md", "src/builtinlibraries/Manual/zh/Welcome.md", "src/builtinlibraries/Manual/zh-HK/Welcome.md", "src/builtinlibraries/Manual/zh-TW/Welcome.md"])
}
```

把相关vertify方法注释掉即可

## 绕过用户注册验证

找到 mainprocess.js

搜索 『vertify』关键字找到下面 class

```js
verifyLicense(e) {
this.getMachineUniqueId(t  => {
if (void  0  !==  t) this.loadLicense(t, (i, r) => {
if (i) return  void  e(i);
let  s  =  r.signature;
void  0  ===  s  && (s  =  r.key);
let  n  =  r.retailer;
void  0  ===  s  && (s  =  ""), void  0  ===  n  && (n  =  "offical");
// 注释前
// let o = this.verify(t, n, s),
// 注释后
let  o  =  true,
a  =  this.settingService.runtime.license;
a.uniqueId  =  t, a.retailer  =  n, a.signature  =  s, a.verified  =  o, e(null, a)
});
else {
let  t  =  this.settingService.runtime.license;
e(null, t)
}
})
}
```

把相关vertify方法注释掉即可
