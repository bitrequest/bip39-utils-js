# BIP39 Utils JS

Standalone JavaScript BIP39/BIP32 HD Wallet library for cryptocurrency applications. Extracted from [bitrequest](https://github.com/bitrequest/bitrequest.github.io) for reuse in other projects.

*No Node.js, no WebAssembly, just vanilla JavaScript.*

## Features

- **Mnemonic Generation** - Generate BIP39 compliant 12/15/18/21/24 word phrases
- **Mnemonic Validation** - Validate checksums and detect invalid words
- **Seed Derivation** - PBKDF2 with 2048 iterations (BIP39 standard)
- **BIP32 Key Derivation** - Hierarchical deterministic key derivation
- **Multi-Currency Support** - Bitcoin, Litecoin, Ethereum, Dogecoin, Dash, Bitcoin Cash
- **Address Formatting** - Legacy (P2PKH), SegWit (Bech32), and CashAddr formats
- **Extended Keys** - xpub/zpub generation and parsing with address derivation
- **Built-in Compatibility Testing** - Verify browser/environment support

## Live Demo

**[🔑 BIP39 Utils Test Suite](https://bitrequest.github.io/unit_tests_bip39_utils.html)** - Interactive tests and tools

**[🔐 Crypto Utils Test Suite](https://bitrequest.github.io/unit_tests_crypto_utils.html)** - Low-level crypto utilities

---

## Download

### Quick Download (Terminal)

```bash
mkdir bip39_utils && cd bip39_utils && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_bip39_utils.html && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_bip39_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js
```

Then open `unit_tests_bip39_utils.html` in your browser.

### Manual Download

| File | Description |
|------|-------------|
| [assets_js_lib_sjcl.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js) | Stanford JavaScript Crypto Library |
| [assets_js_lib_crypto_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js) | Crypto utilities (secp256k1, hashing, encoding) |
| [assets_js_lib_bip39_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_bip39_utils.js) | BIP39/BIP32 HD wallet functions |
| [unit_tests_bip39_utils.html](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_bip39_utils.html) | Interactive test suite |

---

## Usage

### HTML Setup

```html
<script src="assets_js_lib_sjcl.js"></script>
<script src="assets_js_lib_crypto_utils.js"></script>
<script src="assets_js_lib_bip39_utils.js"></script>
```

---

## Quick Start Examples

### Generate Mnemonic

```javascript
// Generate 12-word mnemonic (128-bit entropy)
const mnemonic = Bip39Utils.generate_mnemonic(12);
// "army van defense carry jealous true garbage claim echo media make crunch"

// Other options: 15, 18, 21, or 24 words
const mnemonic24 = Bip39Utils.generate_mnemonic(24);
```

### Validate Mnemonic

```javascript
const mnemonic = "army van defense carry jealous true garbage claim echo media make crunch";

// Check if valid BIP39 phrase (includes checksum validation)
const isValid = Bip39Utils.validate_mnemonic(mnemonic);
// true

// Find invalid words (pass array of words)
const words = mnemonic.split(" ");
const invalidWord = Bip39Utils.find_invalid_word(words);
// undefined (all valid) or returns the first invalid word
```

### Derive Bitcoin SegWit Address

```javascript
const mnemonic = "army van defense carry jealous true garbage claim echo media make crunch";

// Step 1: Convert mnemonic to seed
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);

// Step 2: Get master root key
const rootKey = Bip39Utils.get_rootkey(seed);
const masterKey = rootKey.slice(0, 64);   // First 32 bytes = private key
const chainCode = rootKey.slice(64);       // Last 32 bytes = chain code

// Step 3: Derive key at BIP84 path (SegWit)
const derived = Bip39Utils.derive_x({
    dpath: "m/84'/0'/0'/0/0",  // BIP84 Bitcoin SegWit, first address
    key: masterKey,
    cc: chainCode
});

// Step 4: Generate SegWit address
const publicKey = CryptoUtils.get_publickey(derived.key);
const address = CryptoUtils.pub_to_address_bech32("bc", publicKey);
// "bc1qg0azlj4w2lrq8jssrrz6eprt2fe7f7edm4vpd5"
```

### Derive Bitcoin Legacy Address

```javascript
const mnemonic = "army van defense carry jealous true garbage claim echo media make crunch";
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);
const rootKey = Bip39Utils.get_rootkey(seed);

// Derive at BIP44 path (Legacy)
const derived = Bip39Utils.derive_x({
    dpath: "m/44'/0'/0'/0/0",  // BIP44 Bitcoin Legacy
    key: rootKey.slice(0, 64),
    cc: rootKey.slice(64)
});

// Generate Legacy address using format_keys
const config = Bip39Utils.get_bip32dat("bitcoin");
const keys = Bip39Utils.format_keys(seed, derived, config, 0, "bitcoin");
// { index: 0, address: "1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm", pubkey: "02a1...", privkey: "Kym..." }
```

### Derive Ethereum Address

```javascript
const mnemonic = "army van defense carry jealous true garbage claim echo media make crunch";
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);
const rootKey = Bip39Utils.get_rootkey(seed);
const masterKey = rootKey.slice(0, 64);
const chainCode = rootKey.slice(64);

const derived = Bip39Utils.derive_x({
    dpath: "m/44'/60'/0'/0/0",  // BIP44 Ethereum
    key: masterKey,
    cc: chainCode
});

const config = Bip39Utils.get_bip32dat("ethereum");
const keys = Bip39Utils.format_keys(seed, derived, config, 0, "ethereum");
// { address: "0x...", pubkey: "...", privkey: "..." }
```

---

## Extended Public Keys (xpub/zpub)

Extended public keys allow generating receive addresses without exposing private keys - essential for watch-only wallets and payment processing.

### Generate xpub/zpub

```javascript
const mnemonic = "army van defense carry jealous true garbage claim echo media make crunch";
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);
const rootKey = Bip39Utils.get_rootkey(seed);
const masterKey = rootKey.slice(0, 64);
const chainCode = rootKey.slice(64);

// Derive to account level (m/84'/0'/0' for BTC SegWit)
const derived = Bip39Utils.derive_x({
    dpath: "m/84'/0'/0'",
    key: masterKey,
    cc: chainCode
});

// Get bitcoin config
const config = Bip39Utils.get_bip32dat("bitcoin");

// Generate extended keys
// ext_keys returns { xprv: "zprv...", xpub: "zpub..." }
const extKeys = Bip39Utils.ext_keys(derived, config);

console.log(extKeys.xpub);
// "zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT"

console.log(extKeys.xprv);
// "zprv6rMRRntHrvMM..." (extended private key)
```

### Parse xpub and Derive Addresses

```javascript
const xpub = "zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT";

// Parse xpub to extract key and chaincode
const parsed = Bip39Utils.key_cc_xpub(xpub);
// { key: "0225d06c...", cc: "1975ffb9...", version: "04b24746" }

// Parse extended key details
const decoded = CryptoUtils.b58check_decode(xpub);
const extended = Bip39Utils.objectify_extended(decoded);
// { version: "04b24746", depth: "03", fingerprint: "a49ac50b", 
//   childnumber: "80000000", chaincode: "...", key: "..." }

// Derive child address at index 0 (path M/0/0 from xpub)
// Use "M" (uppercase) for public key derivation
const derived = Bip39Utils.derive_x({
    dpath: "M/0/0",
    key: parsed.key,
    cc: parsed.cc
});

// Generate SegWit address from derived public key
const address = CryptoUtils.pub_to_address_bech32("bc", derived.key);
// "bc1qg0azlj4w2lrq8jssrrz6eprt2fe7f7edm4vpd5"
```

### Batch Address Generation from xpub

```javascript
const xpub = "zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT";
const parsed = Bip39Utils.key_cc_xpub(xpub);

// Generate first 5 receive addresses
for (let i = 0; i < 5; i++) {
    const derived = Bip39Utils.derive_x({
        dpath: "M/0/" + i,  // M/0/0, M/0/1, M/0/2...
        key: parsed.key,
        cc: parsed.cc
    });
    const address = CryptoUtils.pub_to_address_bech32("bc", derived.key);
    console.log(`Address ${i}: ${address}`);
}
```

---

## Supported Currencies

| Currency | Coin ID | SegWit Path | Legacy Path | Address Format |
|----------|---------|-------------|-------------|----------------|
| Bitcoin | `bitcoin` | `m/84'/0'/0'/0/` | `m/44'/0'/0'/0/` | bc1q... / 1... |
| Litecoin | `litecoin` | `m/84'/2'/0'/0/` | `m/44'/2'/0'/0/` | ltc1q... / L... |
| Ethereum | `ethereum` | - | `m/44'/60'/0'/0/` | 0x... |
| Dogecoin | `dogecoin` | - | `m/44'/3'/0'/0/` | D... |
| Dash | `dash` | - | `m/44'/5'/0'/0/` | X... |
| Bitcoin Cash | `bitcoin-cash` | - | `m/44'/145'/0'/0/` | bitcoincash:q... |

### xpub Version Bytes

| Version Hex | Prefix | Used By |
|-------------|--------|---------|
| `04b24746` | zpub | BTC SegWit, LTC SegWit |
| `0488b21e` | xpub | BTC Legacy, BCH, DASH, ETH |
| `019da462` | Ltub | LTC Legacy |
| `02facafd` | dgub | DOGE |

---

## Compatibility Testing

The library includes built-in test functions to verify browser/environment compatibility before use.

### Quick Compatibility Check

```javascript
// Check if BIP39 derivation works
const results = Bip39Utils.test_bip39_compatibility();
if (results.compatible) {
    console.log("Environment compatible!");
}
```

### Individual Test Functions

```javascript
// Test mnemonic → seed derivation
Bip39Utils.test_seed();  // returns true/false

// Test BIP44 address derivation
Bip39Utils.test_derivation();  // returns true/false

// Test xpub address derivation
Bip39Utils.test_xpub_support();  // returns true/false

// Full compatibility check (includes CryptoUtils tests)
Bip39Utils.test_bip39_compatibility();
// Returns: {
//   compatible: true/false,
//   crypto_api: true/false,
//   bigint: true/false,
//   secp256k1: true/false,
//   seed: true/false,
//   derivation: true/false,
//   xpub: true/false,
//   errors: [],
//   timing_ms: 25.3
// }
```

### Test Constants

The library exposes test vectors from the standard [BIP39 test phrase](https://github.com/trezor/python-mnemonic/blob/master/vectors.json):

```javascript
const TC = Bip39Utils.bip39_utils_const;

TC.version          // "1.1.0"
TC.test_phrase      // "army van defense carry jealous true garbage claim echo media make crunch"
TC.expected_seed    // "5b56c417303faa3fcba7e57400e120a0ca83ec5a4fc9ffba757fbe63fbd77a89..."
TC.expected_address // "1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm" (BIP44 m/44'/0'/0'/0/0)
TC.test_xpub        // "xpub6Cy7dUR4ZKF22HEuVq7epRgRsoXfL2MK1RE81CSvp1ZySySoYGXk5PUY9y9Cc5ExpnSwXyimQAsVhyyPDNDrfj4xjDsKZJNYgsHXoEPNCYQ"
```

---

## API Reference

### Mnemonic Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `generate_mnemonic` | `word_count` (12/15/18/21/24) | `string` | Generate random BIP39 mnemonic |
| `validate_mnemonic` | `mnemonic` | `boolean` | Validate BIP39 checksum |
| `find_invalid_word` | `word_array` | `string\|undefined` | Find first invalid word |
| `mnemonic_to_seed` | `mnemonic`, `passphrase?` | `string` (hex) | Convert to 512-bit seed |
| `parse_seed` | `mnemonic`, `passphrase?` | `object` | Prepare PBKDF2 inputs |

### BIP32 Derivation

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `get_rootkey` | `seed` (hex) | `string` (128 hex chars) | Master key via HMAC-SHA512 |
| `derive_x` | `{dpath, key, cc}` | `object` | Derive key at path |
| `derive_child_key` | `key, cc, index, is_public, is_hardened` | `object` | Single child derivation |
| `format_keys` | `seed, data, config, index, coin` | `object` | Format as address/WIF |
| `get_bip32dat` | `coin` | `object\|false` | Get BIP32 config for coin |

### Extended Key Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `ext_keys` | `key_data, config` | `{xprv, xpub}` | Generate extended keys |
| `key_cc_xpub` | `xpub` | `{key, cc, version}` | Parse xpub to components |
| `objectify_extended` | `hex` | `object` | Parse extended key fields |
| `xpub_obj` | `config, path, cc, key` | `object` | Build xpub with ID |

### Derivation Path Format

- Use `m/` prefix for **private key** derivation (hardened paths allowed)
- Use `M/` prefix for **public key** derivation (non-hardened only)
- Hardened indices use `'` suffix: `m/44'/0'/0'`
- Standard paths: `m/purpose'/coin'/account'/change/index`

```javascript
// Private key derivation (from master key)
Bip39Utils.derive_x({ dpath: "m/84'/0'/0'/0/0", key: masterKey, cc: chainCode });

// Public key derivation (from xpub)
Bip39Utils.derive_x({ dpath: "M/0/0", key: pubKey, cc: chainCode });
```

### Testing Functions

| Function | Parameters | Returns | Description |
|----------|------------|---------|-------------|
| `test_seed` | - | `boolean` | Test mnemonic → seed derivation |
| `test_derivation` | - | `boolean` | Test BIP44 address derivation |
| `test_xpub_support` | - | `boolean` | Test xpub address derivation |
| `test_bip39_compatibility` | - | `object` | Full compatibility check with timing |

### Exported Constants

| Property | Type | Description |
|----------|------|-------------|
| `bip39_utils_const` | `object` | Test vectors and version info |
| `bip32_configs` | `object` | BIP32 configuration for all supported coins |

---

## Test Vectors

From [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook):

```javascript
Bip39Utils.test_phrase      // "army van defense carry jealous true garbage claim echo media make crunch"
Bip39Utils.expected_seed    // "5b56c417303faa3fcba7e57400e120a0ca83ec5a4fc9ffba757fbe63fbd77a89a1a3be4c67196f57c39a88b76373733891bfaba16ed27a813ceed498804c0570"
Bip39Utils.expected_address // "1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm"
```

### Expected xpubs for Test Phrase

| Currency | Path | Extended Public Key |
|----------|------|---------------------|
| BTC SegWit | m/84'/0'/0' | `zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT` |
| BTC Legacy | m/44'/0'/0' | `xpub6Cy7dUR4ZKF22HEuVq7epRgRsoXfL2MK1RE81CSvp1ZySySoYGXk5PUY9y9Cc5ExpnSwXyimQAsVhyyPDNDrfj4xjDsKZJNYgsHXoEPNCYQ` |
| LTC SegWit | m/84'/2'/0' | `zpub6rbRn1jy6pNoaXtUZugXAx8VV1zBZs6SqSMq6HgQ5mMqcWMviesiraCEHY545HNMH1brYGnnMs3oyEye7TUqMaUrsb9ZzB9iC3dCLRvceeS` |
| LTC Legacy | m/44'/2'/0' | `Ltub2ZWUsczPPDUFM7hvT9nQYot9zMFz7uemJ5bm6ABcCP4hfidiowXSJHawR4xVSvryHh7rHgyai98aWjnaTfYoGUigezgC4qQZcEBt5ZtymeB` |
| DOGE | m/44'/3'/0' | `dgub8smKeZZ7MYVjDHcmH3LQAA8otmJKKPBBdAjEjuejEu73kysYGV239wLfb3NPxFWoByXhd3Et6ndm66zRY5bby3gQfV55rhphg7156KktXhx` |
| DASH | m/44'/5'/0' | `xpub6CQGNLTBiNDBRYeBXryYBrhr4wpPL51t9MabJzsubea9MZq7A3vohyiUr9rySo9aLyqMb9CAVnWewpcwdYBShcHaPcCUhDhAjuXsrEuRb9a` |
| BCH | m/44'/145'/0' | `xpub6Breb6eRL7Ggnn61fkiLHZkySV2J3ENxLQJgnRy6Kxi4KZTRbyaNbZmhH11oamhr9W7y7mG6j8WP1UtWuiRu5nmAUqpSoqKbFbcftTW982k` |
| ETH | m/44'/60'/0' | `xpub6CadpMraPwst8ZHcjTq6M2FYQQ37kou4LFivCP4dhatoHQjHZgSGpm3eGARnAxnZwfv4U6rar7Zvrnksejd4kwLY2DLGYLdTVdowrQdeWjb` |

### Expected Addresses (Index 0)

| Currency | Path | Address |
|----------|------|---------|
| BTC SegWit | m/84'/0'/0'/0/0 | `bc1qg0azlj4w2lrq8jssrrz6eprt2fe7f7edm4vpd5` |
| BTC Legacy | m/44'/0'/0'/0/0 | `1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm` |
| LTC SegWit | m/84'/2'/0'/0/0 | `ltc1qc64rhsxzmnre94nw4spn6866gqhmcpp05suu3c` |
| LTC Legacy | m/44'/2'/0'/0/0 | `LZakyXotaE29Pehw21SoPuU832UhvJp4LG` |
| ETH | m/44'/60'/0'/0/0 | `0x2161DedC3Be05B7Bb5aa16154BcbD254E9e9eb68` |

---

## Interactive Test Suite

The test suite (`unit_tests_bip39_utils.html`) includes:

### Automated Tests (55+ tests)

**Built-in Library Tests**
- `Bip39Utils.test_seed` - Mnemonic → seed derivation
- `Bip39Utils.test_derivation` - BIP44 address derivation
- `Bip39Utils.test_xpub_support` - xpub address derivation
- `Bip39Utils.test_bip39_compatibility` - Full compatibility check

**Test Constants Validation**
- Verify `test_phrase` produces `expected_seed`
- Verify `expected_seed` produces `expected_address`
- Verify `test_xpub` produces `expected_address`

**Unit Tests**
- Mnemonic generation and validation
- Seed derivation verification
- Key derivation at various paths
- Address format validation
- xpub generation and parsing

### Interactive Tools
1. **Generate Mnemonic** - Create new 12/15/18/21/24 word phrases
2. **Validate Mnemonic** - Check phrase validity, detect invalid words, verify checksum
3. **Mnemonic → Seed** - Convert phrase to hex seed with optional passphrase
4. **Seed → Root Key** - Generate master key from seed
5. **BIP32 Derivation** - Derive keys at custom paths for any supported coin
6. **Batch Derivation** - Generate multiple addresses from mnemonic
7. **Generate xPubs** - Create extended public keys for all supported coins
8. **Parse xPub** - Decode xpub and derive addresses

---

## Security Warning

⚠️ **Never enter real seed phrases on websites or share them with anyone.**

This library is intended for:
- Educational purposes
- Development and testing
- Generating watch-only wallets (xpub derivation)

For production wallets, always use hardware wallets or audited wallet software.

---

## Dependencies

- **sjcl.js** - Stanford JavaScript Crypto Library (SHA, HMAC, PBKDF2)
- **crypto_utils.js** - Low-level crypto operations (secp256k1, Base58, Bech32)

---

## Related Projects

- [crypto-utils-js](https://github.com/bitrequest/crypto-utils-js) - Low-level cryptocurrency utilities
- [xmr-utils-js](https://github.com/bitrequest/xmr-utils-js) - Monero cryptographic utilities
- [bitrequest](https://github.com/bitrequest/bitrequest.github.io) - Cryptocurrency point-of-sale PWA

---

## License

AGPL-3.0 License

## Credits

Developed for [bitrequest.io](https://bitrequest.io) - Lightweight cryptocurrency point-of-sale.
