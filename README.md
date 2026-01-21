# BIP39 Utils

Standalone Javascript BIP39/BIP32 HD Wallet library for cryptocurrency applications. Extracted from [bitrequest](https://github.com/bitrequest/bitrequest.github.io) for reuse in other projects. - no Node.js required

## Features

- **Mnemonic Generation** - Generate BIP39 compliant 12/15/18/21/24 word phrases
- **Mnemonic Validation** - Validate checksums and detect invalid words
- **Seed Derivation** - PBKDF2 with 2048 iterations (BIP39 standard)
- **BIP32 Key Derivation** - Hierarchical deterministic key derivation
- **Multi-Currency Support** - Bitcoin, Litecoin, Ethereum, Dogecoin, Dash, Bitcoin Cash
- **Address Formatting** - Legacy, SegWit (Bech32), and CashAddr formats
- **Extended Keys** - xpub/xprv generation and parsing

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

### Supported Currencies

| Currency | Default Path | Address Format |
|----------|--------------|----------------|
| Bitcoin | `m/84'/0'/0'/0/` | Native SegWit (bc1q...) |
| Litecoin | `m/84'/2'/0'/0/` | Native SegWit (ltc1q...) |
| Ethereum | `m/44'/60'/0'/0/` | 0x... |
| Dogecoin | `m/44'/3'/0'/0/` | D... |
| Dash | `m/44'/5'/0'/0/` | X... |
| Bitcoin Cash | `m/44'/145'/0'/0/` | CashAddr |

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
| `derive_x(params)` | Derive child key at path |
| `key_cc_xpub(xpub)` | Parse xpub to key + chaincode |
| `format_keys(seed, data, config, index, coin)` | Format for currency |
| `get_bip32dat(coin)` | Get BIP32 config |

---

## Test Vectors

From [Mastering Bitcoin](https://github.com/bitcoinbook/bitcoinbook):

```javascript
Bip39Utils.test_phrase     // "army van defense carry jealous true garbage claim echo media make crunch"
Bip39Utils.expected_seed   // "5b56c417303faa3fcba7e57400e120a0..."
Bip39Utils.expected_address // "1HQ3rb7nyLPrjnuW85MUknPekwkn7poAUm"
```

---

## Security Warning

⚠️ **Never enter real seed phrases on websites or share them with anyone.**

---

## License

MIT License

## Credits

Developed for [bitrequest.io](https://bitrequest.io) - Lightweight cryptocurrency point-of-sale.
