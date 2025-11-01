# PyMon Usage Guide

## 🚀 How to Run PyMon Commands

Since PyMon was installed in user space, you have several options to run it:

### Option 1: Python Module (Works Everywhere)
```bash
python -m pymon.cli <command>
```

Examples:
```bash
python -m pymon.cli init my_project
python -m pymon.cli wallet new
python -m pymon.cli compile
python -m pymon.cli deploy SimpleStorage
```

### Option 2: Use the Batch File (In Project Directory)
```bash
.\pymon.bat <command>
```

Examples:
```bash
.\pymon.bat init my_project
.\pymon.bat wallet new
.\pymon.bat compile
.\pymon.bat deploy SimpleStorage
```

### Option 3: Add to PATH (Permanent Solution)

Add this directory to your PATH:
```
C:\Users\nothi\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.13_qbz5n2kfra8p0\LocalCache\local-packages\Python313\Scripts
```

Then you can use `pymon` directly from anywhere.

## ✅ Quick Test

Your project was successfully created! Here's how to continue:

```bash
# Navigate to your project
cd my_project

# Create a wallet (from project directory)
python -m pymon.cli wallet new

# Or using the batch file from parent directory
cd ..
.\pymon.bat wallet new

# Compile contracts
python -m pymon.cli compile

# Deploy to Monad Testnet
python -m pymon.cli deploy SimpleStorage
```

## 📁 Project Structure Created

```
my_project/
├── contracts/
│   ├── SimpleStorage.py    # Python contract
│   └── SimpleStorage.sol   # Solidity contract
├── scripts/
│   └── deploy.py          # Deployment script
├── build/                 # Compiled artifacts (after compile)
└── pymon_config.json      # Monad testnet configuration
```

## 🔗 Next Steps

1. **Get Testnet MON**: Visit https://discord.gg/monaddev
2. **Create Wallet**: `python -m pymon.cli wallet new`
3. **Compile**: `python -m pymon.cli compile`
4. **Deploy**: `python -m pymon.cli deploy SimpleStorage`

## 💡 Pro Tip

Create an alias in your PowerShell profile for easier access:

```powershell
# Add to your PowerShell profile
function pymon { python -m pymon.cli $args }
```

Then you can just use `pymon` directly!
