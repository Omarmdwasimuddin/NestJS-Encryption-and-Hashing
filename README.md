# Encryption and Hashing (NestJS)

**Source:** https://docs.nestjs.com/security/encryption-and-hashing

**Encryption** হলো information কে encode করার process। এই process, information এর original representation — যেটাকে plaintext বলা হয় — সেটাকে ciphertext নামের একটা alternative form এ convert করে। Ideally, শুধু authorized party রাই একটা ciphertext কে আবার plaintext এ decipher করে original information access করতে পারবে। Encryption নিজে থেকে interference আটকায় না, কিন্তু কোনো would-be interceptor এর কাছে content টাকে বোধগম্য হতে দেয় না। Encryption হলো একটা two-way function — যা encrypt করা হয়েছে, সঠিক key দিয়ে সেটা decrypt করা যায়।

**Hashing** হলো একটা নির্দিষ্ট key কে অন্য একটা value তে convert করার process। একটা mathematical algorithm অনুযায়ী নতুন value generate করার জন্য একটা hash function ব্যবহার করা হয়। একবার hashing হয়ে গেলে, output থেকে input এ ফিরে যাওয়া impossible হওয়া উচিত।

---

## 1. Encryption

Node.js এ একটা built-in [crypto module](https://nodejs.org/api/crypto.html) আছে, যেটা দিয়ে string, number, buffer, stream ইত্যাদি encrypt আর decrypt করা যায়। Nest নিজে থেকে এই module এর উপরে আলাদা কোনো package দেয় না — অপ্রয়োজনীয় abstraction avoid করার জন্য।

উদাহরণ হিসেবে, চলো AES (Advanced Encryption System) এর `'aes-256-ctr'` algorithm এর CTR encryption mode ব্যবহার করি।

```typescript
import { createCipheriv, randomBytes, scrypt } from 'node:crypto';
import { promisify } from 'node:util';

const iv = randomBytes(16);
const password = 'Password used to generate key';

// Key এর length algorithm এর উপর নির্ভর করে।
// এই ক্ষেত্রে aes256 এর জন্য, এটা 32 bytes।
const key = (await promisify(scrypt)(password, 'salt', 32)) as Buffer;
const cipher = createCipheriv('aes-256-ctr', key, iv);

const textToEncrypt = 'Nest';
const encryptedText = Buffer.concat([
  cipher.update(textToEncrypt),
  cipher.final(),
]);
```

এবার `encryptedText` value টা decrypt করার জন্য:

```typescript
import { createDecipheriv } from 'node:crypto';

const decipher = createDecipheriv('aes-256-ctr', key, iv);
const decryptedText = Buffer.concat([
  decipher.update(encryptedText),
  decipher.final(),
]);
```

---

## 2. Hashing

Hashing এর জন্য, আমরা [bcrypt](https://www.npmjs.com/package/bcrypt) অথবা [argon2](https://www.npmjs.com/package/argon2) — এই দুটো package এর যেকোনো একটা ব্যবহার করার recommend করি। Nest নিজে থেকে এই module গুলোর উপরে আলাদা কোনো wrapper দেয় না — অপ্রয়োজনীয় abstraction avoid করার জন্য (যাতে learning curve সহজ থাকে)।

উদাহরণ হিসেবে, চলো `bcrypt` ব্যবহার করে একটা random password hash করি।

প্রথমে, প্রয়োজনীয় package গুলো install করো:

```shell
npm i bcrypt
npm i -D @types/bcrypt
```

Installation শেষ হলে, নিচের মতো `hash` function ব্যবহার করতে পারো:

```typescript
import bcrypt from 'bcrypt';

const saltOrRounds = 10;
const password = 'random_password';
const hash = await bcrypt.hash(password, saltOrRounds);
```

একটা salt generate করার জন্য, `genSalt` function ব্যবহার করো:

```typescript
const salt = await bcrypt.genSalt();
```

একটা password compare/check করার জন্য, `compare` function ব্যবহার করো:

```typescript
const isMatch = await bcrypt.compare(password, hash);
```

Available function গুলো সম্পর্কে আরো জানতে [এখানে](https://www.npmjs.com/package/bcrypt) দেখো।
