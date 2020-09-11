# node层数据加密

有的时候我们在做一些分享类的业务时需要带上一些有关端内用户的信息，这个时候一般需要进行加密，最安全的就是在node层去做这个加密
如果是图省事，我们一般可以用atob和btoa进行base64的编码和解码(当然过程中肯定是要加盐的了)，但是即便是这样对于专业的web开发者还是可以用巧妙的方式轻松解密，那么有没有一种方式可以让我们有更多的方案进行数据加密呢？当然，见下文，用crypto的原生方法

```js
const crypto = require('crypto');

const userId = '10086';

function cryptData(algorithm = 'blowfish', okey = '') {
  const iv = Buffer.alloc(8, 0);

  const key = okey || crypto.scryptSync('brelly', 'salt', 10086);

  return {
    encrypt(str = '') {
      const cipher = crypto.createCipheriv(algorithm, key, iv);

      let encrypted = cipher.update(str, 'utf8', 'hex');

      encrypted += cipher.final('hex');

      return encrypted;
    },
    decrypt(str = '') {
      const decipher = crypto.createDecipheriv(algorithm, key, iv);

      let decrypted = decipher.update(str, 'hex', 'utf8');

      decrypted += decipher.final('utf8');

      return decrypted;
    },
  };
}

const crypt = cryptData();

const result = crypt.encrypt(userId);

console.log('result', result);

const origin = crypt.decrypt(result);

console.log('origin', origin);


```


参考链接
https://juejin.im/post/5b17fb165188257d377620d1#heading-8
https://segmentfault.com/a/1190000016706501
https://www.liaoxuefeng.com/wiki/1022910821149312/1023025778520640