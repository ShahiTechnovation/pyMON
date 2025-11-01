# PyMon Wallet Setup Guide

## 🔐 Two Ways to Use Wallets in PyMon

PyMon supports **two wallet methods** for maximum flexibility:

1. **Environment Variable Method** (Quick & Easy)
2. **Encrypted Keystore Method** (Secure & Recommended)

---

## 📋 Method 1: Environment Variable (.env file)

### Best For:
- Development and testing
- Quick prototyping  
- CI/CD pipelines
- When you already have a private key

### Setup Steps:

#### Step 1: Create .env File
```bash
# Create .env from template
python -m pymon.cli env setup

# Or manually
cp .env.example .env
```

#### Step 2: Add Your Private Key
Edit `.env` file and add your private key:
```bash
# .env
PRIVATE_KEY=your_64_character_hex_private_key_here
```

**Important:** 
- Private key should be 64 hex characters
- Can optionally include `0x` prefix
- Never commit .env to git!

#### Step 3: Verify Setup
```bash
python -m pymon.cli env status
```

You should see:
```
✓ .env file found
✓ Private key configured (PRIVATE_KEY)
```

#### Step 4: Deploy
```bash
python -m pymon.cli compile
python -m pymon.cli deploy MyContract
```

PyMon will automatically use the private key from .env!

### Getting a Private Key

If you don't have a private key yet:

**Option A: Export from MetaMask**
1. Open MetaMask
2. Click account menu → Account Details
3. Click "Export Private Key"
4. Enter password
5. Copy private key to .env

**Option B: Create with PyMon**
```bash
# Create new wallet (generates and saves key)
python -m pymon.cli wallet new

# Then export the key and add to .env
# (Key is in pymon_key.json encrypted)
```

---

## 🔒 Method 2: Encrypted Keystore

### Best For:
- Production deployments
- Secure key storage
- When you don't want keys in files
- Team environments

### Setup Steps:

#### Step 1: Generate New Wallet
```bash
python -m pymon.cli wallet new
```

You'll be prompted:
```
Enter password for new wallet: ****
Confirm password: ****
```

#### Step 2: Save the Details
PyMon creates `pymon_key.json` with your encrypted key:
```
✓ New wallet created successfully!

Address: 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb8
Keystore: pymon_key.json

⚠️  Important:
• Keep your password safe - it cannot be recovered
• Back up your keystore file
• Fund your wallet with MON
```

**Save this information securely!**

#### Step 3: Fund Your Wallet
Get testnet MON from Discord:
```bash
# Visit the faucet
https://discord.gg/monaddev

# Request MON for your address:
# 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb8
```

#### Step 4: Deploy
When deploying, PyMon will prompt for your password:
```bash
python -m pymon.cli compile
python -m pymon.cli deploy MyContract
```

---

## 🔄 Wallet Priority System

PyMon checks for wallets in this order:

```
1. PRIVATE_KEY environment variable
   ↓ (if not found)
2. PRIVATE_KEY in .env file
   ↓ (if not found)  
3. pymon_key.json (encrypted keystore)
   ↓ (if not found)
4. avax_key.json (legacy keystore)
   ↓ (if none found)
5. Error: No wallet configured
```

This means:
- .env takes priority over keystore
- You can override keystore by setting PRIVATE_KEY
- Multiple options for flexibility

---

## 📊 Comparison

| Feature | Environment Variable | Encrypted Keystore |
|---------|---------------------|-------------------|
| **Security** | ⚠️ Medium | ✅ High |
| **Ease of Use** | ✅ Very Easy | 🟡 Requires Password |
| **Setup Time** | ⚡ Instant | 🕐 1 minute |
| **Best For** | Development | Production |
| **Password Needed** | ❌ No | ✅ Yes |
| **File to Backup** | `.env` | `pymon_key.json` |
| **CI/CD Friendly** | ✅ Yes | 🟡 Needs Secrets |

---

## 🛠️ Common Workflows

### Workflow 1: Quick Development
```bash
# 1. Setup environment
python -m pymon.cli env setup

# 2. Add private key to .env
echo "PRIVATE_KEY=your_key_here" > .env

# 3. Deploy
python -m pymon.cli deploy Token
```

### Workflow 2: Secure Production
```bash
# 1. Create encrypted wallet
python -m pymon.cli wallet new
# Password: ****

# 2. Fund wallet
# Visit https://discord.gg/monaddev

# 3. Deploy (will prompt for password)
python -m pymon.cli deploy Token
# Password: ****
```

### Workflow 3: Team Development
```bash
# Team Leader:
# 1. Create .env.example with template
# 2. Each developer creates their own .env
# 3. .env is gitignored (never committed)

# Each Developer:
cp .env.example .env
# Edit .env with your PRIVATE_KEY
python -m pymon.cli deploy Token
```

### Workflow 4: CI/CD Pipeline
```bash
# In CI/CD system (GitHub Actions, etc):
# Set PRIVATE_KEY as secret environment variable

# In workflow:
- name: Deploy Contract
  env:
    PRIVATE_KEY: ${{ secrets.PRIVATE_KEY }}
  run: |
    python -m pymon.cli compile
    python -m pymon.cli deploy Token
```

---

## 🔍 Checking Wallet Status

### Check Environment Setup
```bash
python -m pymon.cli env status
```

Shows:
- .env file status
- Wallet configuration
- Network settings
- What's configured and what's missing

### Check Wallet Balance
```bash
python -m pymon.cli wallet show
```

Shows:
- Wallet address
- Source (env or keystore)
- How to check balance on explorer

---

## 📝 Environment File Reference

### Full .env Example
```bash
# Wallet (choose one)
PRIVATE_KEY=your_64_hex_character_private_key

# OR use keystore
# KEYSTORE_FILE=pymon_key.json  
# KEYSTORE_PASSWORD=your_password

# Network
NETWORK=monad-testnet
RPC_URL=https://testnet-rpc.monad.xyz/
CHAIN_ID=10143
EXPLORER_URL=https://testnet.monadexplorer.com/

# Optional: Gas settings
GAS_BUFFER_PERCENT=20
MAX_GAS_PRICE_GWEI=100

# Optional: Debug mode
DEBUG=true
```

---

## 🚨 Security Best Practices

### DO ✅
1. **Use .env for development** - Quick and easy
2. **Use keystore for production** - More secure
3. **Add .env to .gitignore** - Already done in PyMon
4. **Use strong passwords** - For keystore encryption
5. **Backup keystore files** - Store securely
6. **Rotate keys regularly** - Security hygiene
7. **Use hardware wallets** - For large amounts

### DON'T ❌
1. **Never commit .env to git** - Contains private keys
2. **Never share private keys** - Anyone can steal funds
3. **Never use mainnet keys on testnet** - Separation of concerns
4. **Never store passwords in code** - Use secure secret management
5. **Never reuse passwords** - Each wallet should be unique

---

## 🐛 Troubleshooting

### Error: "No private key found"

**Solution:**
```bash
# Check what's configured
python -m pymon.cli env status

# Option 1: Add to .env
python -m pymon.cli env setup
# Then edit .env file

# Option 2: Create keystore
python -m pymon.cli wallet new
```

### Error: "Invalid PRIVATE_KEY format"

**Solution:**
- Private key must be exactly 64 hex characters
- Can optionally include `0x` prefix
- Example: `0x1234567890abcdef...` (64 chars after 0x)

### Error: "PRIVATE_KEY environment variable not set"

**Solution:**
```bash
# Option 1: Add to .env file
echo "PRIVATE_KEY=your_key" >> .env

# Option 2: Set in terminal (temporary)
export PRIVATE_KEY=your_key

# Option 3: Use keystore instead
python -m pymon.cli wallet new
```

### Keystore password forgotten

**Solution:**
Unfortunately, keystore passwords cannot be recovered. You'll need to:
1. Create a new wallet: `python -m pymon.cli wallet new`
2. Transfer any funds from old wallet (if you have backup)
3. Use the new wallet going forward

---

## 📚 Additional Resources

- **Get Testnet MON**: https://discord.gg/monaddev
- **Monad Explorer**: https://testnet.monadexplorer.com/
- **MetaMask Setup**: https://metamask.io/
- **PyMon Documentation**: README.md

---

## ✅ Quick Reference

```bash
# Setup Commands
python -m pymon.cli env setup      # Create .env template
python -m pymon.cli env status     # Check configuration
python -m pymon.cli wallet new     # Create encrypted wallet
python -m pymon.cli wallet show    # Show wallet info

# Deploy with .env method
echo "PRIVATE_KEY=your_key" > .env
python -m pymon.cli deploy Token

# Deploy with keystore method
python -m pymon.cli wallet new     # Create wallet first
python -m pymon.cli deploy Token   # Will prompt for password
```

---

**PyMon - Flexible Wallet Management for Monad Blockchain** 🔐🚀
