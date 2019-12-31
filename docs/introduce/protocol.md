
# 1. 关于 
-----

BIF DID（以下简称BID）方法规范符合W3C社区小组当前发布的DID规范中的要求。有关DID和DID方法规范的更多信息，请参阅[DID入门](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md)和[DID规范](https://w3c-ccg.github.io/did-spec/)。

# 2. 摘要
----

为实体和数字对象等提供分布式标识和身份服务，旨在构建一套去中心化的、权利下放的、数据安全、隐私得到保护的并且好用的、灵活的标识符系统，能够为人、企业、设备和数字对象之间建立可信连接和价值交互，实现万物互联、信任万物的愿景。

BID采用去中心化的身份标识协议，具有去中心化、自主管理、隐私保护、安全易用等特点。每⼀个BID都会对应⼀个BID描述对象（BID DDO）。

# 3. BID 命名空间
-----------

标识此DID方法的method name 是： bid

使用此方法的DID必须以前缀 did:bid 开头。根据DID规范，此字符串必须为小写。在前缀之后的DID的剩余部分由特定算法生成。

# 4. 适用系统
--------

这种DID方法适用于BIF网络，从BIF发布开始正式使用。

# 5. BID标识符
---------

BID方案由以下ABNF定义：

```
bif-did = "did:bid:" idstring *(":" subnamespace)
idstring = 12(base58char)
subnamespace = ALPHA *(ALPHA / DIGIT / "_" / "-")
```

所有BID使用椭圆曲线加密算法随机生成公钥，然后对公钥做哈希值并取前12位加上did:bid组成

举个例子：did:bid:6cc796b8d6e2fbebc9b3cf9e

# 6. BID DDO规范
-----------

BID对应的身份描述对象DDO存储在BIF区块链上，由DDO的控制人写入到区块链，并向所有用户开放读取。

DDO规范包含如下信息：

-   Name:bid标识符昵称，为了提高bid标识符的易用性特别设置，在客户端使用时可用该昵称代替bid标识符

-   publicKey：用户用于身份认证的公钥信息，包括公钥id、公钥类型、公钥对应的BID标识符，授权范围（如果该值为all,最该公钥可对bid文档做任何操作）；

-   authentication：用户身份认证列表信息，包含认证类型type,认证时使用的公钥，可信声明ID

-   Attributes：用户填写的个人信息值（例如：{type:age,value:22}）

-   isEnable: 该BID是否启用

举个例子：

```
{
  "@context": "https://w3id.org/future-method/v1",
  "id": "did:bid:DaogpsaKqKMe4nzPy4Aq",
  "name":"shiweijun",
  "publicKey": [{
    "id": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-1",
    "type": "RsaVerificationKey2018",
    "controller": "did:bid:DaogpsaKqKMe4nzPy4Aq",
    "authorizations": ["all"]，
    "bid": " did:bid:DaogpsaKqKMe4nzPy4Aw"
  }, {
    "id": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-3",
    "type": "Ieee2410VerificationKey2018",
    "controller": "did:bid:DaogpsaKqKMe4nzPy4Aq",
    "bid": " did:bid:DaogpsaKqKMe4nzPy4An"
  }],
  "authentication": [{
    "type": "ED25519SigningAuthentication",
    "publicKey": "did:bid:DaogpsaKqKMe4nzPy4Aq#keys-1",
    "proofId":"c1cb770ac94000d946e583d13ea310"
            }],
  "Attributes":[{
  		"type": "age",
  		"value":21
   }],
   "isEnable":true,
   "createTime":1561945638,
   "updateTime":1561945638
}
```



## 6.1 BID 方法
--------

BIdContract是在BIF区块链平台上BID协议的智能合约实现。借助BIdContract合约，用户可以管理自己的公钥列表、修改自己的个人资料、设置昵称等操作。

### 设置昵称

用户调用SetBidName方法进行设置和更新bid昵称。只有authorizations=all的公钥才能修改

```
bool SetBidName(string name);
```

name:bid昵称，如果name为空，则昵称字段默认为bid标识符

### 查询

查询身份信息,根据bid或者昵称name查询用户的DID文档信息，返回值为DID文档的json串

```
byte[] findDDOByBid(string type,string value)
```

参数：

- ​		type 传入的值的类型 0：bid ,1：name
- ​		Value:传入的值

### 授权

BID支持用户授权控制，即用户可以授权其他bid对当前bid进行控制，例如授权其他bid更新DDO文档等操作。用户调用addBid方法为其他bid授权,只有authorizations=all的公钥才能进行添加

```
 bool addPublicKey(string type,string bid,string authorizations)
```

参数：

- type: 传入的bid产生时所用的加密方法，例如：RSA,ECC
- bid:传入的BID值

- authorizations：为改公钥的授予的权限


### 取消授权

用户调用deleteBid方法取消对其他BID的授权,只有authorizations=all的公钥才能进行取消该授权，获得授权账户不可以自行取消授权

```
 bool deletePublicKey(string bid)
```

参数：bid:传入的BID值

### 添加认证凭证

BID支持用户进行多源身份认证，认证结束后用户可以将可信声明的摘要信息加入DDO文档，便于后续验证和管理。用户调用addProof方法添加可信声明信息,只有authorizations=all的公钥才能进行添加

```
  bool addProof(string type,string proofId)
```

参数：

- type:传入的多源认证的类型,例如：0:基础身份认证（公安认证），1：学历认证，2：护照认证，3：微信认证等
- proofId:可信声明在链上存储的ID编号


### 删除认证凭证

用户调用deleteProof方法删除可信声明信息,只有authorizations=all的公钥才能进行添加

```
bool deleteProof(string proofId)
```

参数：proofId:可信声明在链上存储的ID编号

### 添加属性

用户调用addAttr方法添加个人属性信息

```
bool addAttr(string key,string value)
```

参数：

- type: 传入属性的key,例如：age
- value:传入的属性值


### 删除属性

用户调用deleteAttr方法删除个人属性信息

```
 bool deleteAttr(string type)
```

参数：type: 传入属性的key,例如：age

### 启用

用户调用enable方法启用该bid,只有authorizations=all的公钥才能进行启用

```
  bool enable()
```

### 禁用

用户调用disable方法禁用该bid,只有authorizations=all的公钥才能进行禁用

```
  bool disable()
```
# 7. 安全与隐私保护

## 7.1  安全

BID 方法至少考虑以下安全风险:

* **防止DDOS攻击**

​		BID基于BIF区块链系统，天然防止DDOS攻击

* **防隐私数据窃取攻击**

​		在BID中所有的涉及用户隐私的数据都保存在用户本地，链上以及公开数据只有hash值或者通过其他算法生成的字符串，其他攻击者拿到该数据并不能反推出任何用户隐私数据

* **共识安全**

​        为了确保区块链节点的稳定性，BIF使用DPOS+IBFT双层共识算法

## 7.2 隐私保护

​		为了保护用户隐私数据安全，BID除了将所有涉及隐私数据保存在客户本地客户端外，还在原先公私钥的基础上添加zk-snarks零知识证明算法以及C-L签名算法来加强保护。在各种安全措施保护下，用户隐私数据不会上链以及泄露，上链的部分数据也不会反推出个人信息。
