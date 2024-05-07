# Optimail-One-Click-Unsubscribe

## Background

This repository contains examples on how to verify signature from one-click unsubscribe request to prove authenticity of request.

Let's say system that handles unsubscribes receives the following request:

```
POST /unsubscribe?email=email%40gmail.com&signature=JnKuuIW/5gFtWOl5KvpYBHa53buGSx0WwbeX/kKL98w= HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100
```

Query params:

`email` - url-encoded email recipient value

`signature` - base64 encoded HMAC digested with SHA256 algorithm using the secret key generated in optimove settings e.g. `xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==`

## Code examples to verify signature

### C#
```csharp
static string ComputeHMACSHA256Hash(string message, string base64Key)
{
    byte[] key = Convert.FromBase64String(base64Key);
    byte[] messageBytes = Encoding.UTF8.GetBytes(message);

    using (HMACSHA256 hmac = new HMACSHA256(key))
    {
        byte[] hash = hmac.ComputeHash(messageBytes);
        return Convert.ToBase64String(hash);
    }
}

static bool VerifyHMACSHA256Hash(string email, string base64Key, string signature)
{
    string computedHash = ComputeHMACSHA256Hash(email, base64Key);
    return computedHash == signature;
}
```

### JavaScript
```js
const crypto = require('crypto');

function verifyHMACSHA256Hash(message, base64Key, signature) {
    const key = Buffer.from(base64Key, 'base64');
    const computedHash = crypto.createHmac('sha256', key)
                                .update(message)
                                .digest('base64');

    return computedHash === signature;
}


const isSignatureValid = verifyHMACSHA256Hash("email@gmail.com", "xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==", "JnKuuIW/5gFtWOl5KvpYBHa53buGSx0WwbeX/kKL98w=")
```

### Python
```python
import hashlib
import hmac
import base64

def verify_hmac_sha256_hash(message, base64_key, signature):
    key = base64.b64decode(base64_key)
    computed_hash = hmac.new(key, message.encode('utf-8'), hashlib.sha256).digest()
    computed_hash_base64 = base64.b64encode(computed_hash).decode('utf-8')
    return computed_hash_base64 == signature

is_signature_valid = verify_hmac_sha256_hash("email@gmail.com", "xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==", "JnKuuIW/5gFtWOl5KvpYBHa53buGSx0WwbeX/kKL98w=")
```

### Java
```java
public static boolean verifyHMACSHA256Hash(String message, String base64Key, String signature) {
    try {
        byte[] keyBytes = Base64.getDecoder().decode(base64Key);
        SecretKeySpec keySpec = new SecretKeySpec(keyBytes, "HmacSHA256");
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(keySpec);
        byte[] hash = mac.doFinal(message.getBytes());
        String computedHash = Base64.getEncoder().encodeToString(hash);
        return computedHash.equals(signature);
    } catch (NoSuchAlgorithmException | java.security.InvalidKeyException e) {
        e.printStackTrace();
        return false;
    }
}
```
