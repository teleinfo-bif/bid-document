# 成为信任锚
在整个 BID 解决方案中，Issuer 作为权威机构，主要是用来签发权威声明的。

信任锚需要提供如下功能：

* 当用户向信任锚申请颁发证书时，信任锚能及时应答用户请求颁发证书
* 提供一个吊销列表查询接口，方便对 Issuer 签发的 Claim 进行吊销

在BIF中信任锚主要分为根信任锚以及扩展信任锚。根信任锚是由超级节点投票产生，本质上来说，根信任锚为中心化信任锚，该信任锚只能由超级节点注册管理等。扩展信任锚由用户可随意注册产生。

# 成为根信任锚

# 成为扩展信任锚