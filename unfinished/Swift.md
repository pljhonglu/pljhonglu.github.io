# static

static 关键字修饰的属性的作用域可以在块外，原则上不应该这样


```
class NetworkClient: Manager {
    
    static let defaultClient:NetworkClient = {
        return NetworkClient()
    }()
}


// 调用
NetworkClient.defaultClient
```

原则上 static 修饰的 defaultClient 作用于应该在块内，要想块外访问可以 `static class let defaultClient:NetworkClient = ...`



```
{
	code:
	msg:
	data:{
	
	}
}
```

