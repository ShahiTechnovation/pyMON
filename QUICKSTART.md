# 🚀 PyMon Quick Start Guide

## Installation

```bash
# Clone the repository
git clone https://github.com/ShahiTechnovation/pyMON.git
cd pyMON

# Install dependencies
pip install -r requirements.txt

# Setup environment
python -m pymon.cli env setup
```

## Your First Contract in 5 Minutes

### 1. Initialize Project

```bash
python -m pymon.cli init my_first_contract
cd my_first_contract
```

### 2. Write Contract

Edit `contracts/MyToken.py`:

```python
from pymon.py_contracts import PySmartContract

class MyToken(PySmartContract):
    def __init__(self):
        self.name = "MyToken"
        self.symbol = "MTK"
        self.total_supply = 1000000
        self.balances = {self.msg_sender(): self.total_supply}
    
    @public_function
    def transfer(self, to: str, amount: int):
        sender = self.msg_sender()
        if self.balances.get(sender, 0) < amount:
            raise Exception("Insufficient balance")
        
        self.balances[sender] -= amount
        self.balances[to] = self.balances.get(to, 0) + amount
        self.event("Transfer", sender, to, amount)
    
    @view_function
    def balance_of(self, account: str) -> int:
        return self.balances.get(account, 0)
```

### 3. Audit Contract

```bash
python -m pymon.cli audit contract MyToken
```

### 4. Compile & Deploy

```bash
# Compile
python -m pymon.cli compile

# Deploy to Monad Testnet
python -m pymon.cli deploy MyToken
```

## Get Test MON

1. Join Monad Discord: https://discord.gg/monaddev
2. Use faucet to get test MON
3. Your wallet address: Check with `python -m pymon.cli wallet info`

## Useful Commands

```bash
# Wallet
python -m pymon.cli wallet new       # Create wallet
python -m pymon.cli wallet info      # Show wallet info

# Contracts
python -m pymon.cli compile          # Compile all contracts
python -m pymon.cli audit all        # Audit all contracts
python -m pymon.cli deploy <name>    # Deploy contract

# Environment
python -m pymon.cli env status       # Check environment
```

## Example Contracts

Check the `contracts/` directory for examples:
- `SimpleStorage.py` - Basic storage contract
- `Counter.py` - Counter with increment/decrement
- `PyToken.py` - ERC20-like token
- `VotingContract.py` - Voting system
- `StakingRewards.py` - Staking with rewards

## Next Steps

1. Read the full [README.md](./README.md)
2. Review [Security Audit Guide](./AUDIT_GUIDE.md)
3. Check [Architecture](./ARCHITECTURE.md)
4. Join our community

## Support

- GitHub Issues: https://github.com/ShahiTechnovation/pyMON/issues
- Documentation: Check the docs/ folder
- Examples: Check contracts/ folder

Happy coding! 🐍✨
