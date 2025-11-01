# PyMon Architecture

## 🏗️ System Architecture

### Core Architecture (Python-Native)

```
┌─────────────────────────────────────────────────────────────┐
│                    PyMon Core System                        │
│                  (No Solidity Required)                     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │         Python Contract (.py)         │
        │   from pymon.py_contracts import...   │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │      PyMon Transpiler (Built-in)      │
        │    • Parse Python AST                 │
        │    • Analyze Contract Structure       │
        │    • Generate EVM Bytecode            │
        │    • Create ABI                       │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │         EVM Bytecode + ABI            │
        │      (Ready for Deployment)           │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │          Monad Blockchain             │
        │        (10,000 TPS Network)           │
        └───────────────────────────────────────┘
```

### Optional Solidity Support (If py-solc-x Installed)

```
┌─────────────────────────────────────────────────────────────┐
│              Optional Solidity Path                         │
│          (Only if py-solc-x installed)                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │      Solidity Contract (.sol)         │
        │   pragma solidity ^0.8.19;            │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │     py-solc-x (Optional Module)       │
        │    • Download Solidity Compiler       │
        │    • Compile .sol files               │
        │    • Generate Bytecode + ABI          │
        └───────────────────────────────────────┘
                            │
                            ▼
        ┌───────────────────────────────────────┐
        │         EVM Bytecode + ABI            │
        │      (Ready for Deployment)           │
        └───────────────────────────────────────┘
```

## 📦 Module Structure

```
pymon/
├── __init__.py              # Package initialization
├── cli.py                   # ✅ CLI commands (Python-focused)
├── compiler.py              # ✅ Main compiler (Python-native)
├── transpiler.py            # ✅ Python → EVM transpiler
├── deployer.py              # ✅ Contract deployment
├── wallet.py                # ✅ Wallet management
├── interactor.py            # ✅ Contract interaction
├── py_contracts.py          # ✅ Python contract base class
├── utils.py                 # ✅ Utilities
└── solidity_support.py      # ⚠️  OPTIONAL (py-solc-x wrapper)
```

### Dependency Map

```
Core Dependencies (Always Required):
├── web3 ──────────────► Blockchain interaction
├── typer ─────────────► CLI framework
├── rich ──────────────► Terminal UI
├── eth-account ───────► Wallet/accounts
├── cryptography ──────► Encryption
└── pycryptodome ──────► Additional crypto

Optional Dependencies:
└── py-solc-x ─────────► Solidity compilation (NOT USED by default)
```

## 🔄 Compilation Flow

### Python Contract Flow (Default)

```
1. User writes Python contract
   └─► contracts/Token.py

2. Run: python -m pymon.cli compile
   └─► pymon/cli.py

3. CLI calls compiler
   └─► pymon/compiler.py
       ├─► Finds .py files
       ├─► Reads Python source
       └─► Calls transpiler

4. Transpiler processes Python
   └─► pymon/transpiler.py
       ├─► Parse Python AST
       ├─► Extract state variables
       ├─► Extract functions
       ├─► Generate EVM bytecode
       └─► Generate ABI

5. Save artifacts
   └─► build/Token/
       ├─► Token.json (complete artifact)
       ├─► Token_abi.json (ABI only)
       └─► Token_bytecode.txt (bytecode only)

6. Ready to deploy!
   └─► python -m pymon.cli deploy Token
```

### Solidity Contract Flow (Optional, if py-solc-x installed)

```
1. User has Solidity contract
   └─► contracts/Token.sol

2. Run: python -m pymon.cli compile
   └─► pymon/cli.py

3. CLI calls compiler
   └─► pymon/compiler.py
       ├─► Finds .sol files
       ├─► Checks if py-solc-x available
       └─► If YES: Calls solidity_support.py
           If NO: Shows friendly message

4. Solidity support module
   └─► pymon/solidity_support.py
       ├─► Import solcx
       ├─► Install Solidity compiler
       ├─► Compile .sol file
       └─► Generate bytecode + ABI

5. Save artifacts (same as Python)
   └─► build/Token/...

6. Ready to deploy!
```

## 🎨 CLI Command Flow

### Command: `python -m pymon.cli init my_project`

```
cli.py:init()
    │
    ├─► Create project structure
    │   ├─► my_project/contracts/
    │   ├─► my_project/build/
    │   └─► my_project/scripts/
    │
    ├─► Generate Python contracts
    │   ├─► SimpleStorage.py
    │   └─► Counter.py
    │
    ├─► Create pymon_config.json
    │   └─► Monad Testnet configuration
    │
    └─► Display success message
```

### Command: `python -m pymon.cli compile`

```
cli.py:compile()
    │
    ├─► Load contracts directory
    │
    ├─► Call compiler.compile_contracts()
    │   │
    │   ├─► Find .py files ✅
    │   ├─► Find .sol files (check only)
    │   │
    │   ├─► For each .py file:
    │   │   ├─► Read source code
    │   │   ├─► Call transpiler.transpile_python_contract()
    │   │   ├─► Save artifacts
    │   │   └─► Record result
    │   │
    │   └─► For each .sol file (if py-solc-x available):
    │       ├─► Call solidity_support.compile_solidity_contract()
    │       └─► Save artifacts
    │
    └─► Display results table
```

### Command: `python -m pymon.cli deploy Token`

```
cli.py:deploy()
    │
    ├─► Load pymon_config.json
    │   └─► Monad Testnet RPC, Chain ID
    │
    ├─► Load wallet
    │   └─► wallet.py:WalletManager()
    │
    ├─► Load contract artifacts
    │   └─► compiler.get_contract_artifacts("Token")
    │       ├─► Read build/Token/Token.json
    │       └─► Extract ABI + bytecode
    │
    ├─► Call deployer.deploy_contract()
    │   │
    │   ├─► Connect to Monad RPC (web3.py)
    │   ├─► Check wallet balance (MON)
    │   ├─► Estimate gas
    │   ├─► Build transaction
    │   ├─► Sign with wallet
    │   ├─► Send transaction
    │   ├─► Wait for confirmation
    │   └─► Save to deployments.json
    │
    └─► Display success + contract address
```

## 🔐 Security Architecture

```
Wallet Layer:
├── wallet.py
│   ├─► PBKDF2 encryption
│   ├─► SHA-256 hashing
│   └─► Encrypted keystore (pymon_key.json)
│
├── Environment variable support
│   └─► PRIVATE_KEY (for CI/CD)
│
└── Never stores plain-text keys
```

## 🌐 Network Architecture

```
PyMon Client
    │
    ├─► web3.py (HTTP Provider)
    │   │
    │   └─► Monad RPC
    │       ├─► https://testnet-rpc.monad.xyz/
    │       │   • Chain ID: 10143
    │       │   • Testnet
    │       │
    │       └─► Alternative RPCs:
    │           ├─► https://rpc.ankr.com/monad_testnet
    │           └─► https://rpc-testnet.monadinfra.com
    │
    └─► Explorer (for verification)
        └─► https://testnet.monadexplorer.com/
```

## 📊 Data Flow

### Contract Deployment Data Flow

```
1. Python Source (.py)
   │
   ▼
2. AST (Abstract Syntax Tree)
   │
   ▼
3. Contract Metadata
   ├─► State variables
   ├─► Functions
   ├─► Events
   └─► Constructor args
   │
   ▼
4. EVM Bytecode
   ├─► Initialization code
   └─► Runtime code
   │
   ▼
5. Transaction
   ├─► From: User wallet
   ├─► To: null (contract creation)
   ├─► Data: Bytecode
   └─► Gas: Estimated amount
   │
   ▼
6. Monad Blockchain
   ├─► Validates transaction
   ├─► Executes bytecode
   └─► Assigns contract address
   │
   ▼
7. Deployment Record
   └─► deployments.json
       ├─► Contract name
       ├─► Address
       ├─► Transaction hash
       └─► Block number
```

## 🎯 Key Design Principles

### 1. Python-First
- Primary path uses only Python
- Solidity is optional add-on
- Zero Solidity knowledge required

### 2. Graceful Degradation
- Works without py-solc-x
- Shows helpful messages if Solidity files found
- Suggests Python alternatives

### 3. Simple Installation
- Minimal required dependencies
- Optional extras for advanced features
- Fast installation without Solidity compiler

### 4. Clear Separation
- Core in compiler.py (Python)
- Optional in solidity_support.py (Solidity)
- No mixing of concerns

### 5. Monad Optimized
- Built for 10,000 TPS
- Optimized gas estimation
- Parallel execution aware

## 🚀 Performance Characteristics

### Compilation Speed
- Python contracts: **Fast** (< 1 second per contract)
- Solidity contracts (if used): Slower (3-5 seconds per contract)

### Deployment Speed
- Network latency: 1-2 seconds (Monad's 1-second blocks)
- Gas optimization: Built-in optimizer
- Transaction confirmation: < 2 seconds

### Resource Usage
- Memory: < 50MB for Python-only
- Memory: ~200MB if py-solc-x active
- Disk: < 10MB for Python-only
- Disk: ~60MB if py-solc-x installed

---

## 📝 Summary

PyMon architecture is designed around **Python-native smart contract development**:

✅ **Core Path**: Python → Transpiler → EVM → Monad  
⚠️ **Optional Path**: Solidity → py-solc-x → EVM → Monad

The system works perfectly without py-solc-x, making it accessible to Python developers without Solidity knowledge!
