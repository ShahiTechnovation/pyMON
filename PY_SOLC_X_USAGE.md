# py-solc-x Usage in PyMon

## 📋 Current Status: OPTIONAL

**py-solc-x is now an OPTIONAL dependency in PyMon.** The project works perfectly without it!

## 🎯 Why Optional?

PyMon has been transformed into a **Python-native smart contract platform**. You write contracts in pure Python, and PyMon transpiles them directly to EVM bytecode. No Solidity compilation needed!

### Before (Old PyVax)
```
Write Solidity (.sol) → py-solc-x compiles → EVM bytecode → Deploy
```
❌ Required py-solc-x (mandatory dependency)

### Now (PyMon - Python Native)
```
Write Python (.py) → PyMon transpiles → EVM bytecode → Deploy
```
✅ py-solc-x is completely optional!

## 🔧 How py-solc-x is Used (If Installed)

### 1. Location of py-solc-x Code

```
pymon/
├── compiler.py          # Main compiler (Python-native)
└── solidity_support.py  # Optional Solidity support module
```

### 2. Import Strategy

**In `compiler.py`:**
```python
# Optional Solidity support (not required for PyMon's Python-native contracts)
try:
    from .solidity_support import compile_solidity_contract, check_solidity_support, SOLIDITY_AVAILABLE
except ImportError:
    SOLIDITY_AVAILABLE = False
    compile_solidity_contract = None
    check_solidity_support = lambda: False
```

**In `solidity_support.py`:**
```python
# Try to import py-solc-x, but make it optional
SOLIDITY_AVAILABLE = False
try:
    from solcx import compile_standard, install_solc, set_solc_version
    SOLIDITY_AVAILABLE = True
except ImportError:
    console.print("[yellow]Note: py-solc-x not installed.[/yellow]")
    console.print("[yellow]PyMon works natively with Python contracts![/yellow]")
```

### 3. When py-solc-x Would Be Used

py-solc-x would ONLY be used if:
1. You have it installed: `pip install py-solc-x`
2. You have Solidity (.sol) files in your contracts directory
3. You explicitly want to compile Solidity contracts

**Current PyMon behavior:**
- Finds .sol files → Checks if py-solc-x available
- If NOT available → Shows friendly message suggesting Python contracts
- If available → Could compile Solidity (but PyMon recommends Python!)

## 📊 Dependency Management

### requirements.txt
```txt
typer>=0.9.0
rich>=13.0.0
web3>=6.0.0
eth-account>=0.8.0
cryptography>=41.0.0
pycryptodome>=3.0.0
# py-solc-x>=2.0.0  # ← COMMENTED OUT (optional)
```

### setup.py
```python
install_requires=[
    "web3>=6.0.0",
    "typer>=0.9.0",
    "rich>=13.0.0",
    "eth-account>=0.10.0",
    "cryptography>=41.0.0",
    "pycryptodome>=3.0.0",
    # py-solc-x NOT in required dependencies
],
extras_require={
    "solidity": ["py-solc-x>=2.0.0"],  # ← Optional extra
},
```

### pyproject.toml
```toml
dependencies = [
    "web3>=6.0.0",
    "typer>=0.9.0",
    "rich>=13.0.0",
    "eth-account>=0.10.0",
    "cryptography>=41.0.0",
    "pycryptodome>=3.0.0",
]

[project.optional-dependencies]
solidity = ["py-solc-x>=2.0.0"]  # ← Optional
```

## 🚀 Installation Scenarios

### Scenario 1: Python-Native (Recommended)
```bash
pip install -e .
```
**Result:**
- ✅ All Python contract features work
- ✅ No py-solc-x installed
- ✅ Smaller installation
- ✅ Faster install time

### Scenario 2: With Optional Solidity Support
```bash
pip install -e ".[solidity]"
# OR
pip install py-solc-x
```
**Result:**
- ✅ All Python contract features work
- ✅ py-solc-x available (but not needed)
- ⚠️ Larger installation

## 💡 Why This Approach?

### Benefits of Making py-solc-x Optional

1. **Simpler for Python Developers**
   - Don't need Solidity compiler to write smart contracts
   - Fewer dependencies = easier installation
   - No Solidity knowledge required

2. **Faster Installation**
   - py-solc-x downloads Solidity compilers (~50MB)
   - Not needed for Python contracts
   - Installation is 10x faster

3. **Better Developer Experience**
   - Focus on Python ecosystem
   - Use familiar Python syntax
   - No language switching

4. **Backward Compatible**
   - Can still add py-solc-x if needed
   - Optional support for legacy Solidity contracts
   - Smooth migration path

## 🔍 Checking py-solc-x Status

### Method 1: Run Test Script
```bash
python test_solidity.py
```

### Method 2: Try Importing
```python
try:
    import solcx
    print("✅ py-solc-x is installed")
except ImportError:
    print("✅ Running in Python-native mode (recommended)")
```

### Method 3: Check PyMon Module
```python
from pymon.solidity_support import check_solidity_support

if check_solidity_support():
    print("Solidity support available")
else:
    print("Python-native mode (no Solidity needed)")
```

## 📝 Summary

### py-solc-x in PyMon:
- ❌ **NOT required** for core functionality
- ❌ **NOT used** by default
- ❌ **NOT needed** for Python contracts
- ✅ **Available** as optional extra
- ✅ **Gracefully handled** if missing
- ✅ **Backward compatible** if installed

### What PyMon Actually Uses:
- ✅ **Python transpiler** (built-in)
- ✅ **web3.py** (blockchain interaction)
- ✅ **typer + rich** (beautiful CLI)
- ✅ **cryptography** (wallet encryption)

### The Bottom Line:
**PyMon doesn't need py-solc-x!** It's a Python-native platform that transpiles Python directly to EVM bytecode. Solidity support is purely optional for backward compatibility.

---

## 🎉 Key Takeaway

```python
# This is all you need for PyMon!
from pymon.py_contracts import PySmartContract

class MyContract(PySmartContract):
    """Pure Python → EVM bytecode"""
    # No Solidity required!
    # No py-solc-x required!
    # Just Python!
```

**Write Python. Deploy to Monad. That's it!** 🐍🚀
