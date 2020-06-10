# noble-bls12-381 ![Node CI](https://github.com/paulmillr/noble-secp256k1/workflows/Node%20CI/badge.svg) [![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)

bls12-381, a pairing-friendly Barreto-Lynn-Scott elliptic curve construction. Allows to:

- Construct [zk-SNARKs](https://z.cash/technology/zksnarks/) at the 128-bit security
- Use [threshold signatures](https://medium.com/snigirev.stepan/bls-signatures-better-than-schnorr-5a7fe30ea716),
  which allows a user to sign lots of messages with one signature and verify them swiftly in a batch,
  using Boneh-Lynn-Shacham signature scheme.

**The fastest implementation written in a scripting language**. Matches following specs:

- [Pairing-friendly curves 02](https://tools.ietf.org/html/draft-irtf-cfrg-pairing-friendly-curves-02)
- [BLS signatures 02](https://tools.ietf.org/html/draft-irtf-cfrg-bls-signature-02)
- [Hash to curve 07](https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-07)

Check out [BLS12-381 For The Rest Of Us](https://hackmd.io/@benjaminion/bls12-381) to get started with the primitives.

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies
- Easily auditable TypeScript/JS code
- Uses es2020 bigint. Supported in Chrome, Firefox, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [bls12-381](https://github.com/paulmillr/noble-bls12-381),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160)

## Usage

Node.js and browser:

> npm install noble-bls12-381

```js
import * as bls from "bls12-381";

// Use hex or Uint8Arrays
const privateKey = '67d53f170b908cabb9eb326c3c337762d59289a8fec79f7bc9254b584b73265c';
const msg = 'hello';

(async () => {
  const publicKey = bls.getPublicKey(privateKey);
  const signature1 = await bls.sign(msg, PRIprivateKeyVATE_KEY);
  const isCorrect1 = await bls.verify(msg, publicKey, signature);

  // Sign 1 msg with 3 keys
  const privateKeys = [
    '18f020b98eb798752a50ed0563b079c125b0db5dd0b1060d1c1b47d4a193e1e4',
    'ed69a8c50cf8c9836be3b67c7eeff416612d45ba39a5c099d48fa668bf558c9c',
    '16ae669f3be7a2121e17d0c68c05a8f3d6bef21ec0f2315f1d7aec12484e4cf5'
  ];
  const signatures = await Promise.all(privateKeys.map(p => bls.sign(msg, p)));
  const aggPubKey = await bls.aggregatePublicKeys(publicKeys);
  const aggSignature = await bls.aggregateSignatures(signatures);
  const isCorrect2 = await bls.verify(signature, msg, aggPubKey);

  // Sign 3 msgs with 3 keys
  const messages = ['whatsup', 'all good', 'thanks'];
  const publicKeys = privateKeys.map(bls.getPublicKey);
  const signatures2 = await Promise.all(privateKeys.map((p, i) => bls.sign(messages[i], p)));
  const aggSignature2 = await bls.aggregateSignatures(signatures);
  const isCorrect3 = await bls.verifyMultiple(signature, messages, publicKeys);
})();
```

Deno:

```typescript
import * as bls from "https://deno.land/x/bls12_381/mod.ts";
const publicKey = bls.getPublicKey("18f020b98eb798752a50ed0563b079c125b0db5dd0b1060d1c1b47d4a193e1e4");
```

## API

- [`getPublicKey(privateKey)`](#getpublickeyprivatekey)
- [`sign(message, privateKey)`](#signmessage-privatekey)
- [`verify(signature, hash, publicKey)`](#verifysignature-hash-publickey)
- [`aggregatePublicKeys(publicKeys)`](#aggregatepublickeyspublickeys)
- [`aggregateSignatures(signatures)`](#aggregatesignaturessignatures)
- [`verifyMultiple(hashes, publicKeys, signature)`](#verifymultiplehashes-publickeys-signature)
- [`pairing(G1Point, G2Point)`](#pairingg1point-g2point)

##### `getPublicKey(privateKey)`
```typescript
function getPublicKey(privateKey: Uint8Array | string | bigint): Uint8Array;
```
- `privateKey: Uint8Array | string | bigint` will be used to generate public key.
  Public key is generated by executing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
- Returns `Uint8Array`: encoded publicKey for signature verification

##### `sign(message, privateKey)`
```typescript
function sign(
  message: Uint8Array | string,
  privateKey: Uint8Array | string | bigint
): Promise<Uint8Array>;
```
- `message: Uint8Array | string` - message which would be hashed & signed
- `privateKey: Uint8Array | string | bigint` - private key which will sign the hash
- Returns `Uint8Array`: encoded signature

Default domain (DST) is `BLS_SIG_BLS12381G2_XMD:SHA-256_SSWU_RO_NUL_`, use `bls.DST` to change it.

##### `verify(signature, hash, publicKey)`
```typescript
function verify(
  signature: Uint8Array | string,
  hash: Uint8Array | string,
  publicKey: Uint8Array | string
): Promise<boolean>
```
- `hash: Uint8Array | string` - message hash that needs to be verified
- `publicKey: Uint8Array | string` - e.g. that was generated from `privateKey` by `getPublicKey`
- `signature: Uint8Array | string` - object returned by the `sign` or `aggregateSignatures` function
- Returns `Promise<boolean>`: `true` / `false` whether the signature matches hash

##### `aggregatePublicKeys(publicKeys)`
```typescript
function aggregatePublicKeys(publicKeys: Uint8Array[] | string[]): Uint8Array;
```
- `publicKeys: Uint8Array[] | string[]` - e.g. that have been generated from `privateKey` by `getPublicKey`
- Returns `Uint8Array`: one aggregated public key which calculated from public keys

##### `aggregateSignatures(signatures)`
```typescript
function aggregateSignatures(signatures: Uint8Array[] | string[]): Uint8Array;
```
- `signatures: Uint8Array[] | string[]` - e.g. that have been generated by `sign`
- Returns `Uint8Array`: one aggregated signature which calculated from signatures

##### `verifyMultiple(hashes, publicKeys, signature)`
```typescript
function verifyMultiple(
  hashes: Uint8Array[] | string[],
  publicKeys: Uint8Array[] | string[],
  signature: Uint8Array | string
): Promise<boolean>
```
- `hashes: Uint8Array[] | string[]` - messages hashes that needs to be verified
- `publicKeys: Uint8Array[] | string[]` - e.g. that were generated from `privateKeys` by `getPublicKey`
- `signature: Uint8Array | string` - object returned by the `aggregateSignatures` function
- Returns `Promise<boolean>`: `true` / `false` whether the signature matches hashes

##### `pairing(G1Point, G2Point)`
```typescript
function pairing(
  g1Point: Point<bigint>,
  g2Point: Point<[bigint, bigint]>,
  withFinalExponent: boolean = true
): Point<[bigint, bigint, bigint, bigint, bigint, bigint, bigint, bigint, bigint, bigint, bigint, bigint]>
```
- `g1Point: Point<bigint>` - simple point, `x, y` are bigints
- `g2Point: Point<[bigint, bigint]>` - point over curve with imaginary numbers (`(x, x_1), (y, y_1)`)
- `withFinalExponent: boolean` - should the result be powered by curve order. Very slow.
- Returns `Point<BigintTwelve>`: paired point over 12-degree extension field.

##### Helpers

```typescript
// 𝔽p
bls.CURVE.P // 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaabn

// Prime order
bls.CURVE.r // 0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001n

// Hash base point (x, y)
bls.CURVE.Gx // 0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001n
// x = 3685416753713387016781088315183077757961620795782546409894578378688607592378376318836054947676345821548104185464507
// y = 1339506544944476473020471379941921221584933875938349620426543736416511423956333506472724655353366534992391756441569

// Signature base point ((x_1, x_2), (y_1, y_2))
bls.CURVE.Gy
// x = 3059144344244213709971259814753781636986470325476647558659373206291635324768958432433509563104347017837885763365758, 352701069587466618187139116011060144890029952792775240219908644239793785735715026873347600343865175952761926303160
// y = 927553665492332455747201965776037880757740193453592970025027978793976877002675564980949289727957565575433344219582, 1985150602287291935568054521177171638300868978215655730859378665066344726373823718423869104263333984641494340347905

// Classes
bls.Fq
bls.Fq2
bls.Fq12
bls.G1Point
bls.G2Point
bls.G12Point
```

## Internals

- BLS Relies on Bilinear Pairing (expensive)
- Private Keys: 32 bytes
- Public Keys: 48 bytes: 381 bit affine x coordinate, encoded into 48 big-endian bytes.
- Signatures: 96 bytes: two 381 bit integers (affine x coordinate), encoded into two 48 big-endian byte arrays.
    - The signature is a point on the G2 subgroup, which is defined over a finite field
    with elements twice as big as the G1 curve (G2 is over Fq2 rather than Fq. Fq2 is analogous to the complex numbers).
- The 12 stands for the Embedding degree.

Formulas:

- `P = pk x G` - public keys
- `S = pk x H(m)` - signing
- `e(P, H(m)) == e(G,S)` - verification using pairings
- `e(G, S) = e(G, SUM(n)(Si)) = MUL(n)(e(G, Si))` - signature aggregation

## Speed

**The fastest implementation written in a scripting language** (js, python etc). Has all possible optimizations, like:

- cyclotomatic exponentation
- frobenius coefficients
- endomorphism for clearing cofactor

Benchmarks:

```
getPublicKey x 1080 ops/sec @ 925μs/op
sign x 14 ops/sec @ 70ms/op
aggregateSignatures x 201 ops/sec @ 4ms/op
verify x 22 ops/sec @ 44ms/op
pairing (batch) x 54 ops/sec @ 18ms/op
pairing (single) x 48 ops/sec @ 20ms/op
```

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is potentially vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Use low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.

## Contributing

1. Clone the repository.
2. `npm install` to install build dependencies like TypeScript
3. `npm run compile` to compile TypeScript code
4. `npm run test` to run jest on `test/index.ts`

Special thanks to [Roman Koblov](https://github.com/romankoblov), who have helped to improve pairing speed.

## License

MIT (c) Paul Miller [(https://paulmillr.com)](https://paulmillr.com), see LICENSE file.
