## 前端安全在工作中的实践

### 密码传输

#### 场景

业务中，需要用户将设备的账号、密码传给后端进行校验，进行设备操作。

#### 分析

尽管在请求中，都是采用https的方式进行数据获取的，但是如果明文传输密码还是有很大风险的。

#### 方案

* 对称加密：一套秘钥，解析速度快
* 非对称加密：公(私)钥加密，私(公)钥解密，解析速度慢，成本高

#### 结论

采用对称加密算法(AES)，以用户名作为key的部分、与后端协议好key的具体格式，进行数据传输。

```
import crypto from 'crypto-js';
// 加密
function encryptData(data: string, key: string) {
	const tmp = crypto.AES.encrypt(data, `sea-${key}-salt`);
	return tmp.toString(crypto.enc.Utf8);
}

// 解密
function decryptData(data: string, key: string) {
	const tmp = crypto.AES.decrypt(data, `sea-${key}-salt`);
	return tmp.toString(crypto.enc.Utf8);
}

formData = {
	username: '',
	password: ''
}

const tmp = encryptData(this.formData.password, this.formData.username);
axios.post('/submit', {data: {
	...this.formData,
	password: tmp
}})
```



