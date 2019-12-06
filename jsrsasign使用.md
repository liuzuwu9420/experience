# Token


### 1.1 下载jsrsasign
  ```shell
  npm install jsrsasign -s
  ```
### 1.2 使用jsrsasign   
#### 1.2.1 首先在需要使用的文件中引入
  ```javascript
  import Jsrsasign from 'jsrsasign'
  ```

## Token验证

### 2.1 验证的方法  
  ```javascript
  <static> {Boolean} KJUR.jws.JWS.verifyJWT(sJWT, key, acceptField)
  ```
  
* 参数：  
`{String} sJWT  `  
JSON Web令牌（JWT）的字符串以进行验证  
`{Object} key`  
要验证的公共密钥，证书或密钥对象的字符串  
`{Array} acceptField`  
可接受字段的关联数组（OPTION）  
`Returns:`  
{Boolean} 如果JWT令牌有效，则为true；否则为false 

#### 2.1.1 vue中使用案例
  ```javascript
  var isValid = Jsrsasign.KJUR.jws.JWS.verifyJWT('eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJzeXN0ZW0iLCJzdWIiOiJ7XCJVc2VybmFtZVwiOlwicm9vdFwifSIsImV4cCI6MTU3Mjk0NDExMywiaWF0IjoxNTcyOTQwNTEzLCJqdGkiOiJjYTVhN2Q3Ny02Mjk2LTRlYjctYTg5Ni0yZWY4NWNhM2ExZjcifQ.sS0vMZ5u3o6FxTQJrx5HKiv18o0S8rDgRVP6b1uerAI', { b64: 'MTIzNDU2Nzg=' }, { alg: ['HS256'], gracePeriod: 10 })
  ```
  判断`isValid`为true或false即可  

* 注  
  - 本项目中`key`可以为`{ utf8: '12345678' }` 和 `{ b64: 'MTIzNDU2Nzg=' }`两种验证格式，其中utf8是由后端设计，b64为格式。
  - 因JWT生成器或验证器的时钟可以快也可以慢。如果这些时钟相差很大，则JWT验证可能会失败。为了避免这种情况，`jsrsasign`支持`gracePeriod`参数，该参数指定这些时钟的可接受时间差（以秒为单位）。因此，如果您想在2小时内接受慢速或快速，则可以指定gracePeriod = 2 * 60 * 60;。`默认情况下，gracePeriod为零`。

## Token解析

### 3.1 验证的方法  
  ```javascript
  <static> {Array} KJUR.jws.JWS.parse(sJWT)
  ```
* 解析JWS签名的标头和有效载荷  
此方法解析JWS签名字符串。所得的关联数组具有以下属性：  
`headerObj`-标头的JSON对象  
`payloadObj`-有效载荷的JSON对象，如果有效载荷是JSON字符串，否则未定义  
`headerPP`-通过stringify漂亮打印的JSON标头  
`payloadPP`-通过字符串化（如果有效载荷为JSON则通过字符串化）漂亮打印的JSON有效载荷，否则Base64URL解码的有效载荷原始字符串  
`sigHex`-签名的十六进制字符串  

## Token生成

### 4.1 生成的方法  
  ```javascript
  <static> {String} KJUR.jws.JWS.sign(alg, spHead, spPayload, key, pass)
  ```
    
* 参数：  
`{String} alg  `  
要签名的JWS算法名称并将其强制设置为sHead或null   
`{String} spHead`  
JWS标头的字符串或对象  
`{String} spPayload`  
JWS有效负载的字符串或对象  
`{String} key`
要签名的私钥或mac密钥对象的字符串  
`{String} pass`
（OPTION）密码以使用加密的非对称私钥  
`Returns:`  
{String} JWS签名字符串 

#### 4.1.1 vue中使用案例
  ```javascript
  // Header
var oHeader = { alg: 'HS256', typ: 'JWT' }
// Payload
var oPayload = {}
var tNow = Jsrsasign.KJUR.jws.IntDate.get('now')
var tEnd = Jsrsasign.KJUR.jws.IntDate.get('now + 1day')
oPayload.nbf = tNow
oPayload.iat = tNow
oPayload.exp = tEnd
var sHeader = JSON.stringify(oHeader)
var sPayload = JSON.stringify(oPayload)
var sJWT = Jsrsasign.KJUR.jws.JWS.sign('HS256', sHeader, sPayload, { b64: 'MTIzNDU2Nzg=' })
  ```
  sJWT即为生成的Token

 #### 另，更多使用请查询jsrsasign官网API：
  ```javascript
  https://kjur.github.io/jsrsasign/api/
  ```