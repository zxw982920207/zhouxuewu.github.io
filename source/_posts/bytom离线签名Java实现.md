---

title: Bytom离线签名Java实现
date: 2018-06-16
tags: [blockchain,java]

---

### 整体离线签名实现流程

- 通过一级私钥生成一级公钥（改进的Ed25519)
- 一级私钥生成二级私钥（Hmac-sha512）
- 二级私钥生成扩展私钥（Hmac-sha512）[**扩展私钥才是真正被签名的主体**]
- 对签名的msg进行hash（SHA3-256)
- 对扩展私钥和hash之后的msg做加密（改进的Ed25519）

### 1. 二级私钥的生成

- 将字符N、xpub的前32位字节、path数组的第i个全字符拼接在一起，做Hmac-sha512加密

  ```java
  // begin build data
  ByteArrayOutputStream out = new ByteArrayOutputStream();
  out.write('N');
  out.write(xpub, 0, xpub.length / 2);
  out.write(path, 0, path.length);
  byte[] data = out.toByteArray();
  // end build data
  
  // begin build key
  byte[] key = new byte[xpub.length / 2];
  System.arraycopy(xpub, xpub.length / 2, key, 0, xpub.length / 2);
  // end build key
  
  // doFinal()
  byte[] res = HMacSha512(data, key);
  ```

- 将上一步结果的后32位字节进行移位与或操作，得到中间结果

  ```java
  //begin operate res[:32]
  byte[] f = new byte[res.length / 2];
  System.arraycopy(res, 0, f, 0, res.length / 2);
  f = pruneIntermediateScalar(f);
  System.arraycopy(f, 0, res, 0, res.length / 2);
  //end operate res[:32]
  
  //begin operate res[:32] again
  int carry = 0;
  int sum = 0;
  for (int i = 0; i < 32; i++) {
      int xprvInt = xprv[i] & 0xFF;
      int resInt = res[i] & 0xFF;
      sum = xprvInt + resInt + carry;
      res[i] = (byte) sum;
      carry = sum >> 8;
  }
  if ((sum >> 8) != 0) {
      System.err.println("sum does not fit in 256-bit int");
  }
  //end operate res[:32] again
  ```

- 将上一步的结果作为私钥，继续遍历path数组，循环以上两个操作，直到遍历完path数组

  ```java
  byte[][] paths = new byte[][]{
                  Hex.decode(hpaths[0]),
                  Hex.decode(hpaths[1])
          };
  byte[] res = xprv;
  for (int i = 0; i < hpaths.length; i++) {
      byte[] xpub = DeriveXpub.deriveXpub(res);
      res = NonHardenedChild.NHchild(paths[i], res, xpub);
  }
  ```

- 得到的最终结果即为**二级私钥**

### 2. 一级公钥的生成

一级公钥的生成主要依赖`Ed25519.scalarMultWithBaseToBytes(scalar);`方法，一级公钥的前32个字节是私钥的前32个字节Ed25519加密之后的，后32个字节实际上就是私钥的后32个字节。

```java
public class DeriveXpub {
    public static byte[] deriveXpub(byte[] xprv) {
        byte[] xpub = new byte[xprv.length];
        byte[] scalar = new byte[xprv.length / 2];
//        for (int i = 0; i < xprv.length / 2; i++) {
//            scalar[i] = xprv[i];
//        }
        System.arraycopy(xprv, 0, scalar, 0, xprv.length / 2);
        byte[] buf = Ed25519.scalarMultWithBaseToBytes(scalar);
//        for (int i = 0; i < buf.length; i++) {
//            xpub[i] = buf[i];
//        }
        System.arraycopy(buf, 0, xpub, 0, buf.length);
//        for (int i = xprv.length / 2; i < xprv.length; i++) {
//            xpub[i] = xprv[i];
//        }
        System.arraycopy(xprv, xprv.length / 2, xpub, xprv.length / 2, xprv.length / 2);
        return xpub;
    }
}
```

### 3. 扩展私钥的生成

扩展私钥的生成实际上是将`Expand`字符串和二级私钥作`Hmac-sha512`加密之后得到的新的扩展私钥。

```java
public class ExpandedPrivateKey {
    private static byte[] HMacSha512(byte[] data, byte[] key)
            throws SignatureException, NoSuchAlgorithmException, InvalidKeyException
    {
        SecretKeySpec signingKey = new SecretKeySpec(key, "HmacSHA512");
        Mac mac = Mac.getInstance("HmacSHA512");
        mac.init(signingKey);
        return mac.doFinal(data);
    }

    public static byte[] ExpandedPrivateKey(byte[] data)
            throws SignatureException, NoSuchAlgorithmException, InvalidKeyException
    {
        // "457870616e64" is "Expand" hex.
        byte[] res = HMacSha512(data, Hex.decode("457870616e64"));
        for (int i = 0; i <= 31; i++) {
            res[i] = data[i];
        }
//        System.arraycopy(data, 0, res, 0, 32);
        return res;
    }
}
```

### 4. 签名msg的Hash生成

将交易的`input_id`和`tx_id`用sha3-256的算法进行hash

```java
public byte[] hashFn(byte[] hashedInputHex, byte[] txID) {

        SHA3.Digest256 digest256 = new SHA3.Digest256();

        // data = hashedInputHex + txID
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        out.write(hashedInputHex, 0, hashedInputHex.length);
        out.write(txID, 0, txID.length);
        byte[] data = out.toByteArray();

        return digest256.digest(data);
    }
```



### 5. 最终签名结果sig的生成

最终签名的结果是参考比原go源码，用java实现了改进后的`Ed25519`加密算法。

```java
public static byte[] Ed25519InnerSign(byte[] privateKey, byte[] message)
            throws SignatureException, NoSuchAlgorithmException, InvalidKeyException {
        MessageDigest md = MessageDigest.getInstance("SHA-512");
        byte[] digestData = new byte[32 + message.length];
        int digestDataIndex = 0;
        for (int i = 32; i < 64; i++) {
            digestData[digestDataIndex] = privateKey[i];
            digestDataIndex++;
        }
        for (int i = 0; i < message.length; i++) {
            digestData[digestDataIndex] = message[i];
            digestDataIndex++;
        }
        md.update(digestData);
        byte[] messageDigest = md.digest();

        com.google.crypto.tink.subtle.Ed25519.reduce(messageDigest);
        byte[] messageDigestReduced = Arrays.copyOfRange(messageDigest, 0, 32);
        byte[] encodedR = com.google.crypto.tink.subtle.Ed25519.scalarMultWithBaseToBytes(messageDigestReduced);
        byte[] publicKey = DeriveXpub.deriveXpub(privateKey);

        byte[] hramDigestData = new byte[32 + encodedR.length + message.length];
        int hramDigestIndex = 0;
        for (int i = 0; i < encodedR.length; i++) {
            hramDigestData[hramDigestIndex] = encodedR[i];
            hramDigestIndex++;
        }
        for (int i = 0; i < 32; i++) {
            hramDigestData[hramDigestIndex] = publicKey[i];
            hramDigestIndex++;
        }
        for (int i = 0; i < message.length; i++) {
            hramDigestData[hramDigestIndex] = message[i];
            hramDigestIndex++;
        }
        md.reset();
        md.update(hramDigestData);
        byte[] hramDigest = md.digest();
        com.google.crypto.tink.subtle.Ed25519.reduce(hramDigest);
        byte[] hramDigestReduced = Arrays.copyOfRange(hramDigest, 0, 32);

        byte[] sk = Arrays.copyOfRange(privateKey, 0, 32);
        byte[] s = new byte[32];
        com.google.crypto.tink.subtle.Ed25519.mulAdd(s, hramDigestReduced, sk, messageDigestReduced);

        byte[] signature = new byte[64];
        for (int i = 0; i < encodedR.length; i++) {
            signature[i] = encodedR[i];
        }
        int signatureIndex = 32;
        for (int i = 0; i < s.length; i++) {
            signature[signatureIndex] = s[i];
            signatureIndex++;
        }
        return signature;
    }
```



参考：

[Bytom 源代码· GitHub](https://github.com/Bytom)

[tink/Ed25519.java at master · google/tink · GitHub](https://github.com/google/tink/blob/master/java/src/main/java/com/google/crypto/tink/subtle/Ed25519.java)