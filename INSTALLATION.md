# PyMon Installation Guide

## 🚀 Quick Install (Python-Native)

PyMon works out of the box with Python contracts - no Solidity required!

```bash
# Clone the repository
git clone https://github.com/yourusername/pymon
cd pymon

# Install PyMon (Python-native mode)
pip install -e .
```

That's it! You can now write and deploy Python smart contracts.

## 📦 Installation Options

### Option 1: Python-Native Only (Recommended)

This is the default installation. PyMon will work with Python contracts only.

```bash
pip install -e .
```

**What you get:**
- ✅ Full Python smart contract support
- ✅ Direct Python to EVM transpilation
- ✅ All PyMon features
- ✅ No Solidity dependencies
- ✅ Simpler, cleaner installation

### Option 2: With Optional Solidity Support

If you need backward compatibility with Solidity contracts:

```bash
# Install with Solidity support
pip install -e ".[solidity]"

# Or install py-solc-x separately
pip install py-solc-x
```

**What you get:**
- ✅ Everything from Python-native mode
- ✅ Optional Solidity compilation
- ⚠️ Larger installation size
- ⚠️ More dependencies

## 🐍 Why Python-Native?

PyMon is designed to work **natively with Python** contracts:

1. **No Solidity Required** - Write contracts in Python you already know
2. **Faster Development** - No need to learn a new language
3. **Better Errors** - Python error messages are clearer
4. **Simpler Setup** - Fewer dependencies, faster installation

## 📋 Dependencies

### Core Dependencies (Always Required)
- `web3>=6.0.0` - Blockchain interaction
- `typer>=0.9.0` - CLI framework
- `rich>=13.0.0` - Beautiful terminal output
- `eth-account>=0.10.0` - Wallet management
- `cryptography>=41.0.0` - Encryption
- `pycryptodome>=3.0.0` - Additional crypto utilities

### Optional Dependencies
- `py-solc-x>=2.0.0` - Only if you need Solidity support

## 🔍 Checking Your Installation

After installation, verify everything works:

```bash
# Check PyMon is installed
python -m pymon.cli --help

# Check Python contract support (always available)
python -m pymon.cli init test_project
cd test_project
python -m pymon.cli compile

# Check if Solidity support is available (optional)
python -c "from pymon.solidity_support import check_solidity_support; print('Solidity:', check_solidity_support())"
```

## 🛠️ Troubleshooting

### Issue: "py-solc-x not installed" warning

**Solution:** This is normal! PyMon works perfectly without it.
- If you only use Python contracts (recommended): Ignore this message
- If you need Solidity: `pip install py-solc-x`

### Issue: Command not found

**Solution:** Use the Python module syntax:
```bash
# Instead of: pymon init my_project
# Use: python -m pymon.cli init my_project
```

### Issue: Import errors

**Solution:** Reinstall PyMon:
```bash
pip uninstall pymon
pip install -e .
```

## 🎯 Quick Start After Installation

```bash
# 1. Initialize a project
python -m pymon.cli init my_dapp

# 2. Navigate to project
cd my_dapp

# 3. Write Python contracts (no Solidity needed!)
# contracts/Token.py already created for you

# 4. Compile Python to EVM bytecode
python -m pymon.cli compile

# 5. Deploy to Monad
python -m pymon.cli deploy Token
```

## 📚 What's Next?

- Read [PYMON_PYTHON_NATIVE.md](PYMON_PYTHON_NATIVE.md) to learn about Python contracts
- Check [README.md](README.md) for full documentation
- Join our Discord for support

---

**PyMon - Making Blockchain Development Pythonic!** 🐍🚀
