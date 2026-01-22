# BIP39 Utils

Standalone Javascript BIP39/BIP32 HD Wallet library for cryptocurrency applications. Extracted from [bitrequest](https://github.com/bitrequest/bitrequest.github.io) for reuse in other projects. - no Node.js required

## Features

- **Mnemonic Generation** - Generate BIP39 compliant 12/15/18/21/24 word phrases
- **Mnemonic Validation** - Validate checksums and detect invalid words
- **Seed Derivation** - PBKDF2 with 2048 iterations (BIP39 standard)
- **BIP32 Key Derivation** - Hierarchical deterministic key derivation
- **Multi-Currency Support** - Bitcoin, Litecoin, Ethereum, Dogecoin, Dash, Bitcoin Cash
- **Address Formatting** - Legacy, SegWit (Bech32), and CashAddr formats
- **Extended Keys** - xpub/zpub generation and parsing with address derivation

## Live Demo

**[🔑 BIP39 Utils Test Suite](https://bitrequest.github.io/unit_tests_bip39_utils.html)** - Interactive tests and tools

**[🔐 Crypto Utils Test Suite](https://bitrequest.github.io/unit_tests_crypto_utils.html)** - Low-level crypto utilities

---

## Download

### Quick Download (Terminal)

Run this command to download all required files:

```bash
mkdir bip39_utils && cd bip39_utils && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_bip39_utils.html && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_bip39_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js
```

Then open `unit_tests_bip39_utils.html` in your browser.

### Crypto Utils Only

```bash
mkdir crypto_utils && cd crypto_utils && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_crypto_utils.html && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js && \
curl -O https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js
```

### Manual Download

Right-click and "Save As" for each file:

| File | Description |
|------|-------------|
| [assets_js_lib_sjcl.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_sjcl.js) | Stanford JavaScript Crypto Library |
| [assets_js_lib_crypto_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_crypto_utils.js) | Crypto utilities (secp256k1, hashing, encoding) |
| [assets_js_lib_bip39_utils.js](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/assets_js_lib_bip39_utils.js) | BIP39/BIP32 HD wallet functions |
| [unit_tests_bip39_utils.html](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_bip39_utils.html) | Interactive test suite |
| [unit_tests_crypto_utils.html](https://raw.githubusercontent.com/bitrequest/bitrequest.github.io/master/unit_tests_crypto_utils.html) | Crypto utils test suite |

---

## Usage

### HTML Setup

```html
<script src="assets_js_lib_sjcl.js"></script>
<script src="assets_js_lib_crypto_utils.js"></script>
<script src="assets_js_lib_bip39_utils.js"></script>
```

### Generate Mnemonic

```javascript
const mnemonic = Bip39Utils.generate_mnemonic(12);
// "army van defense carry jealous true garbage claim echo media make crunch"
```

### Validate Mnemonic

```javascript
const isValid = Bip39Utils.validate_mnemonic(mnemonic);
// true

const invalidWord = Bip39Utils.find_invalid_word(mnemonic.split(" "));
// undefined (all valid) or returns the invalid word
```

### Derive Address

```javascript
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);
const rootKey = Bip39Utils.get_rootkey(seed);

const derived = Bip39Utils.derive_x({
    dpath: "m/44'/0'/0'/0/0",
    key: rootKey.slice(0, 64),
    cc: rootKey.slice(64)
});

const config = Bip39Utils.get_bip32dat("bitcoin");
const keys = Bip39Utils.format_keys(seed, derived, config, 0, "bitcoin");
// { index: 0, address: "1HQ3...", pubkey: "02a1...", privkey: "Kym..." }
```

### Generate Extended Public Key (xpub/zpub)

```javascript
const seed = Bip39Utils.mnemonic_to_seed(mnemonic);
const rootKey = Bip39Utils.get_rootkey(seed);
const masterKey = rootKey.slice(0, 64);
const chainCode = rootKey.slice(64);

// Derive to account level (e.g., m/84'/0'/0' for BTC SegWit)
const derived = Bip39Utils.derive_x({
    dpath: "m/84'/0'/0'",
    key: masterKey,
    cc: chainCode
});

// Get public key from derived private key
const publicKey = CryptoUtils.get_publickey(derived.key);

// Build xpub data
const keyData = {
    chaincode: derived.chaincode,
    purpose: "84'",
    childnumber: derived.childnumber,
    depth: derived.depth,
    fingerprint: derived.fingerprint,
    xpub: true,
    key: publicKey
};

// Generate xpub string
const config = Bip39Utils.get_bip32dat("bitcoin");
const extKeys = Bip39Utils.ext_keys(keyData, config);
console.log(extKeys.ext_key);
// "zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT"
```

### Parse xpub and Derive Addresses

```javascript
const xpub = "zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT";

// Parse xpub to get key and chaincode
const parsed = Bip39Utils.key_cc_xpub(xpub);
// { key: "0225d06c...", cc: "1975ffb9...", version: "04b24746" }

// Parse extended key details
const extended = Bip39Utils.objectify_extended(CryptoUtils.b58check_decode(xpub));
// { version, depth, fingerprint, childnumber, chaincode, key }

// Derive child address at index 0 (path M/0/0)
const derived = Bip39Utils.derive_x({
    dpath: "M/0/0",  // Use "M" for public key derivation
    key: parsed.key,
    cc: parsed.cc
});

// Generate address from derived public key
const address = CryptoUtils.pub_to_address_bech32("bc", derived.key);
// "bc1qcr8te4kr609gcawutmrza0j4xv80jy8z306fyu"
```

### Supported Currencies

| Currency | Default Path | Address Format | xpub Format |
|----------|--------------|----------------|-------------|
| Bitcoin SegWit | `m/84'/0'/0'/0/` | Native SegWit (bc1q...) | zpub |
| Bitcoin Legacy | `m/44'/0'/0'/0/` | P2PKH (1...) | xpub |
| Litecoin SegWit | `m/84'/2'/0'/0/` | Native SegWit (ltc1q...) | zpub |
| Litecoin Legacy | `m/44'/2'/0'/0/` | P2PKH (L...) | Ltub |
| Ethereum | `m/44'/60'/0'/0/` | 0x... | xpub |
| Dogecoin | `m/44'/3'/0'/0/` | D... | dgub |
| Dash | `m/44'/5'/0'/0/` | X... | xpub |
| Bitcoin Cash | `m/44'/145'/0'/0/` | CashAddr | xpub |

### xpub Version Bytes

| Version | Format | Coins |
|---------|--------|-------|
| `04b24746` | zpub | BTC SegWit, LTC SegWit |
| `0488b21e` | xpub | BTC Legacy, LTC Legacy, BCH, DASH, DOGE, ETH |
| `019da462` | Ltub | LTC Legacy |
| `02facafd` | dgub | DOGE |

---

## API Reference

### Mnemonic Functions

| Function | Description |
|----------|-------------|
| `generate_mnemonic(word_count)` | Generate random mnemonic (12/15/18/21/24 words) |
| `validate_mnemonic(mnemonic)` | Validate BIP39 checksum |
| `find_invalid_word(word_array)` | Find first invalid word |
| `mnemonic_to_seed(mnemonic, passphrase?)` | Convert to 512-bit seed |

### BIP32 Derivation

| Function | Description |
|----------|-------------|
| `get_rootkey(seed)` | Master key from seed (HMAC-SHA512) |
| `derive_x(params)` | Derive child key at path (use `m/` for private, `M/` for public) |
| `derive_child_key(key, cc, index, is_public, is_hardened)` | Single child derivation |
| `format_keys(seed, data, config, index, coin)` | Format keys for currency |
| `get_bip32dat(coin)` | Get BIP32 config |

### Extended Key Functions

| Function | Description |
|----------|-------------|
| `ext_keys(key_data, config)` | Generate xprv/xpub from key data |
| `xpub_obj(config, path, cc, key)` | Build xpub object with ID |
| `key_cc_xpub(xpub)` | Parse xpub to key + chaincode |
| `objectify_extended(hex)` | Parse extended key components |
| `b58c_x_payload(key_data, config)` | Build Base58Check payload |

---

## Test Vectors

From [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook):

```javascript
Bip39Utils.test_phrase     // "army van defense carry jealous true garbage claim echo media make crunch"
Bip39Utils.expected_seed   // "5b56c417303faa3fcba7e57400e120a0..."
Bip39Utils.expected_address // "1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm"
```

### Expected xpubs for Test Phrase

| Currency | Path | xpub |
|----------|------|------|
| BTC SegWit | m/84'/0'/0' | `zpub6rMRRntHrvMM2EPvnrZ7HmbnNr74JAxmCXFbAwfjNKqGXJbHve9yWMk9SzsF9jQHM6RsaExfmAjyypn369i5dT3uXrRyiDra5nqpCjuPmWT` |
| BTC Legacy | m/44'/0'/0' | `xpub6Cy7dUR4ZKF22HEuVq7epRgRsoXfL2MK1RE81CSvp1ZySySoYGXk5PUY9y9Cc5ExpnSwXyimQAsVhyyPDNDrfj4xjDsKZJNYgsHXoEPNCYQ` |
| LTC SegWit | m/84'/2'/0' | `zpub6rbRn1jy6pNoaXtUZugXAx8VV1zBZs6SqSMq6HgQ5mMqcWMviesiraCEHY545HNMH1brYGnnMs3oyEye7TUqMaUrsb9ZzB9iC3dCLRvceeS` |
| DOGE | m/44'/3'/0' | `dgub8smKeZZ7MYVjDHcmH3LQAA8otmJKKPBBdAjEjuejEu73kysYGV239wLfb3NPxFWoByXhd3Et6ndm66zRY5bby3gQfV55rhphg7156KktXhx` |
| DASH | m/44'/5'/0' | `xpub6CQGNLTBiNDBRYeBXryYBrhr4wpPL51t9MabJzsubea9MZq7A3vohyiUr9rySo9aLyqMb9CAVnWewpcwdYBShcHaPcCUhDhAjuXsrEuRb9a` |
| BCH | m/44'/145'/0' | `xpub6Breb6eRL7Ggnn61fkiLHZkySV2J3ENxLQJgnRy6Kxi4KZTRbyaNbZmhH11oamhr9W7y7mG6j8WP1UtWuiRu5nmAUqpSoqKbFbcftTW982k` |
| ETH | m/44'/60'/0' | `xpub6CadpMraPwst8ZHcjTq6M2FYQQ37kou4LFivCP4dhatoHQjHZgSGpm3eGARnAxnZwfv4U6rar7Zvrnksejd4kwLY2DLGYLdTVdowrQdeWjb` |

---

## Security Warning

⚠️ **Never enter real seed phrases on websites or share them with anyone.**

---

## License

MIT License

## Credits

Developed for [bitrequest.io](https://bitrequest.io) - Lightweight cryptocurrency point-of-sale.
