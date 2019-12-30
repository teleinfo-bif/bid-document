# 1.DID解析器

  在W3C Spec https://w3c-ccg.github.io/did-resolution 中定义了 DID Resolver的主要功能：

* resolve：根据DID返回Document

* dereference：根据DID URL返回所需资源

  

  例如当DID URL中包含service字段，则进行service endpoint url的解析，如下所示：

  ![did-url-dereferencing](../image/did-url-dereferencing.png)

  `注：该图引用自DID解析协议`
  
  # 2. BID DID解析器

### 2.1 bid 解析器地址

```
http://did.bidspace.cn/{bid}
```

### 2.2 bid 解析器API 介绍

**请求参数**：bid

例如：

```
get http://did.bidspace.cn/did:bid:6cc796b8d6e2fbebc9b3cf9e
```

**返回参数**

| 序号 | 参数名称 | 返回值     | 备注                                                         |
| ---- | -------- | ---------- | ------------------------------------------------------------ |
| 1    | code     | 200        | 操作码：200-请求成功、400-业务异常、<br />403-请求被拒绝、500-服务器异常 |
| 2    | success  | true/false | 请求是否成功                                                 |
| 3    | data     | ddo文档    | 返回数据                                                     |
| 4    | msg      | 返回信息   | 返回信息                                                     |

返回信息实例：

```json
{
  "code": 200,
  "success": true,
  "data": {
	{
		'@context': 'https://w3id.org/future-method/v1',
		'Id': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
		'name': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
		'type': '1',
		'publicKeys': [{
			'id': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
			'keyId': 'did:bid:6cc796b8d6e2fbebc9b3cf9e#key-1',
			'type': 'secp256k1',
			'controller': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
			'authority': 'all',
			'publicKey': '04862d66d1255c7ac9c6d86920081069a4d7c295c838140217644ac6f4fcddd0d2edef63951148affb107223fbd1331363e275125bc0cfeb34b1daad26328f8b20'
		}, {
			'id': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
			'keyId': 'did:bid:6cc796b8d6e2fbebc9b3cf9e#key-2',
			'type': 'secp256k1',
			'controller': 'did:bid:6cc796b8d6e2fbebc9b3cf9e',
			'authority': 'base',
			'publicKey': '237755f59dfa66f8b1a55b8212824bd1bad707fd65e85c41975721deab7cf081389eced53a02d164dff39813857a132053458d5b6914b7c793659f9629756d46fd'
		}],
		'authentications': [{
			'proofId': '047755f59dfa66f8b1a55b8212824bd1bad707fd65e85c41975721deab7cf081389eced53a02d164dff39813857a132053458d5b6914b7c793659f9629756d46fc',
			'type': '0000000000000000000000006469643a6269643a',
			'publicKey': '04862d66d1255c7ac9c6d86920081069a4d7c295c838140217644ac6f4fcddd0d2edef63951148affb107223fbd1331363e275125bc0cfeb34b1daad26328f8b20'
		}],
		'attributes': [{
			'attrType': 'name',
			'value': '史维君'
		}],
		'isEnable': '1',
		'createTime': '2019-12-23 03:25:31.039756917 +0000 UTC m=+399791.414534234',
		'updateTime': '2019-12-23 03:25:31.039757176 +0000 UTC m=+399791.414534430'
	}
},
  "msg": ""
}
```

