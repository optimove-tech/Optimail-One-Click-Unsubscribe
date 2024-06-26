# One-Click Unsubscribe

## Background

In order to handle RFC-8058 compliant 1-click unsubscribes from Gmail and Yahoo your server must have a path to accept unauthenticated POST requests which can be configured in your Optimail settings.

Optimail will include unsubscribe links into the headers of your email messages identifying the customer and using a shared secret allowing you to verify that a request is authentic and was generated by Optimove.

This repository contains examples on how to verify the signature from an unsubscribe request.

## Prerequisites

Assuming that your Optimail settings have been configured with an `HTTP/S Unsubscribe` path of `https://mydomain.com/unsubscribe` then the generated unsubscribe links will append querystring parameters for

`email` - url-encoded email recipient value

`signature` - url encoded and base64 encoded HMAC digested with SHA256 algorithm using the secret key generated in optimove settings e.g. `xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==`

an example request you could receive is:

```
POST https://mydomain.com/unsubscribe?email=email%40gmail.com&signature=JnKuuIW%2F5gFtWOl5KvpYBHa53buGSx0WwbeX%2FkKL98w%3D HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100
```

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

static bool VerifySignature(string email, string secretKey, string signature)
{
    string computedHash = ComputeHMACSHA256Hash(email, secretKey);
    return CryptographicOperations.FixedTimeEquals(Convert.FromBase64String(computedHash), Convert.FromBase64String(signature));
}
```

### JavaScript
```js
const crypto = require('crypto');

function verifySignature(email, secretKey, signature) {
    const key = Buffer.from(secretKey, 'base64');
    const computedHash = crypto.createHmac('sha256', key)
                                .update(email)
                                .digest('base64');
    const computedHashBuffer = Buffer.from(computedHash);
    const signatureBuffer = Buffer.from(signature);
    return crypto.timingSafeEqual(computedHashBuffer, signatureBuffer);
}

const isSignatureValid = verifySignature("email@gmail.com", "xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==", "JnKuuIW/5gFtWOl5KvpYBHa53buGSx0WwbeX/kKL98w=")
```

### Python
```python
import hashlib
import hmac
import base64

def verify_signature(email, secret_key, signature):
    key = base64.b64decode(secret_key)
    computed_hash = hmac.new(key, email.encode('utf-8'), hashlib.sha256).digest()
    computed_hash_base64 = base64.b64encode(computed_hash).decode('utf-8')
    return hmac.compare_digest(computed_hash_base64, signature)

is_signature_valid = verify_signature("email@gmail.com", "xnkdrtS59fi9w72EbxtygjQJUJdjFkO+eyTv02sqgjD27yZHivtFUAlqPtkWZnuVVT7SF6T2XiE5bmdWPmALbw==", "JnKuuIW/5gFtWOl5KvpYBHa53buGSx0WwbeX/kKL98w=")
```

### Java
```java
public static boolean verifySignature(String email, String secretKey, String signature) {
    try {
        byte[] keyBytes = Base64.getDecoder().decode(secretKey);
        SecretKeySpec keySpec = new SecretKeySpec(keyBytes, "HmacSHA256");
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(keySpec);
        byte[] hash = mac.doFinal(email.getBytes());
        String computedHash = Base64.getEncoder().encodeToString(hash);
        return computedHash.equals(signature);
    } catch (NoSuchAlgorithmException | java.security.InvalidKeyException e) {
        e.printStackTrace();
        return false;
    }
}
```
