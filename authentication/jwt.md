# jwt

**jwt (Json Web Token)**， 客户端向服务器提供用于登录验证的信息（用户名、密码），服务器签发token，之后客户端向服务器请求资源时带上这个token。服务器根据客户端传来的token验证客户端的身份，返回所请求的资源。

客户端请求时一般会把token放到请求头里。

## 组成

jwt主要由header、playload、signature三部分组成，BASE64加密后的header、BASE加密后的playload和signature用字符.连接起来形成一个jwt。由于BASE64是可逆的，所以jwt里的信息是公开可见的。

        eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

### header

header里保存着类型和所使用的加密算法。

    { "type": "jwt", "alg": "sha256" }
        
### playload

playload里存放在有效信息，可以使用标准的注册声明（不强制），也可以根据业务需要定制自己的声明。

标注注册声明

- **iss**: jwt签发者
- **sub**: jwt所面向的用户
- **aud**: 接收jwt的一方
- **exp**: jwt的过期时间，这个过期时间必须要大于签发时间
- **nbf**: 定义在什么时间之前，该jwt都是不可用的
- **iat**: jwt的签发时间
- **jti**: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击

        { "usrname": "mike", "userId": 46 }

### signature

signature是一段签名，用于检测jwt是否是可信任的。由base64加密后的header和base64加密后的playload用字符.连接后的形成一个字符串，使用header里声明的加密算法和secret对这个字符串进行加密，便的到了signature。

secret保存在服务器，并且必须确保其不被泄露。

## 验证

服务器收到客户端传来的token后，根据token里的header和playload计算出一个signature，如果这个signature根token上的signature相吻合，是可信任且没经过篡改的。

## 优缺点

基于Session认证的一些弊端：

- 由于服务器储存在服务器端，当用户量增多时，服务器的开销会有明显的增大。
- Session基于浏览器Cookie实现，会存在CRSF问题。
- 当服务器进行集群部署时，需要对多个服务器之间进行Session共享。

jwt没有不存在上述问题，但有个缺点：jwt一旦签发后，在有效期内服务器无法对其撤销。
