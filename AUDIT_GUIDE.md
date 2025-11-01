# 🔍 Smart Contract Security Audit Guide

## Overview

PyMon now includes a built-in **Smart Contract Security Auditor** that automatically checks your Python smart contracts for common vulnerabilities and security issues before deployment.

## Features

The auditor checks for:

### 🔴 **Critical Issues**
- **Reentrancy Attacks** - State changes after external calls
- **Unprotected Functions** - Critical functions without access control

### 🟠 **High Severity**
- **Access Control** - Missing authorization checks
- **Denial of Service** - External calls in loops
- **Integer Overflow** - Unchecked arithmetic operations

### 🟡 **Medium Severity**
- **Timestamp Dependence** - Using block.timestamp for critical logic
- **Unsafe Randomness** - Predictable random number generation
- **Gas Issues** - Unbounded loops that could run out of gas

### 🟢 **Low Severity**
- **Input Validation** - Missing parameter checks
- **Hardcoded Values** - Hardcoded addresses or values
- **Gas Optimization** - Inefficient storage operations

### 🔵 **Informational**
- **Missing Events** - State changes without event emission
- **Best Practices** - Code quality improvements

## Usage

### Audit a Single Contract

```bash
python -m pymon.cli audit contract <contract_name>
```

Example:
```bash
python -m pymon.cli audit contract VotingContract
```

### Audit All Contracts

```bash
python -m pymon.cli audit all
```

### Save Audit Report

```bash
python -m pymon.cli audit contract <contract_name> --output report.json
```

## Understanding the Report

### Security Score

- **100/100** - Perfect, no issues found
- **80-99** - Good, minor issues only
- **60-79** - Fair, some medium issues
- **40-59** - Poor, high severity issues
- **0-39** - Critical, do not deploy

### Pass/Fail Criteria

A contract **passes** if:
- ✅ No critical vulnerabilities
- ✅ Maximum 1 high severity issue
- ✅ Score >= 60

A contract **fails** if:
- ❌ Any critical vulnerabilities found
- ❌ Multiple high severity issues
- ❌ Score < 60

## Common Vulnerabilities & Fixes

### 1. Reentrancy Attack

**Vulnerable Code:**
```python
def withdraw(self, amount):
    if self.balance >= amount:
        msg.sender.transfer(amount)  # External call
        self.balance -= amount        # State change after call
```

**Fixed Code:**
```python
def withdraw(self, amount):
    if self.balance >= amount:
        self.balance -= amount        # State change first
        msg.sender.transfer(amount)  # External call last
```

### 2. Missing Access Control

**Vulnerable Code:**
```python
def set_owner(self, new_owner):
    self.owner = new_owner  # Anyone can change owner!
```

**Fixed Code:**
```python
def set_owner(self, new_owner):
    if msg.sender != self.owner:
        raise Exception("Only owner can change owner")
    self.owner = new_owner
```

### 3. Integer Overflow

**Vulnerable Code:**
```python
def add_balance(self, amount):
    self.balance += amount  # Could overflow
```

**Fixed Code:**
```python
def add_balance(self, amount):
    new_balance = self.balance + amount
    if new_balance < self.balance:  # Overflow check
        raise Exception("Integer overflow")
    self.balance = new_balance
```

### 4. Missing Input Validation

**Vulnerable Code:**
```python
def transfer(self, to, amount):
    self.balances[msg.sender] -= amount
    self.balances[to] += amount
```

**Fixed Code:**
```python
def transfer(self, to, amount):
    if amount <= 0:
        raise Exception("Invalid amount")
    if to == address(0):
        raise Exception("Invalid recipient")
    if self.balances[msg.sender] < amount:
        raise Exception("Insufficient balance")
    
    self.balances[msg.sender] -= amount
    self.balances[to] += amount
```

### 5. Gas Optimization

**Inefficient Code:**
```python
def sum_balances(self):
    total = 0
    for user in self.users:  # Unbounded loop
        total += self.balances[user]
    return total
```

**Optimized Code:**
```python
def sum_balances(self, start, count):
    total = 0
    end = min(start + count, len(self.users))
    for i in range(start, end):  # Bounded loop
        total += self.balances[self.users[i]]
    return total
```

## Best Practices

### ✅ **Always Audit Before Deployment**
```bash
# Compile
python -m pymon.cli compile

# Audit
python -m pymon.cli audit contract MyContract

# Deploy only if audit passes
python -m pymon.cli deploy MyContract
```

### ✅ **Use Events for State Changes**
```python
def transfer(self, to, amount):
    # ... transfer logic ...
    self.event("Transfer", msg.sender, to, amount)
```

### ✅ **Implement Access Control**
```python
def only_owner(self):
    if msg.sender != self.owner:
        raise Exception("Not authorized")

@only_owner
def critical_function(self):
    # Protected function
```

### ✅ **Validate All Inputs**
```python
def set_value(self, value):
    if value < 0 or value > MAX_VALUE:
        raise Exception("Invalid value")
    self.value = value
```

### ✅ **Follow Checks-Effects-Interactions Pattern**
```python
def withdraw(self, amount):
    # 1. Checks
    if amount <= 0:
        raise Exception("Invalid amount")
    if self.balance < amount:
        raise Exception("Insufficient balance")
    
    # 2. Effects (state changes)
    self.balance -= amount
    
    # 3. Interactions (external calls)
    msg.sender.transfer(amount)
```

## Integration with CI/CD

### GitHub Actions Example

```yaml
name: Smart Contract Audit

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
    
    - name: Install PyMon
      run: |
        pip install -r requirements.txt
        
    - name: Audit Contracts
      run: |
        python -m pymon.cli audit all
```

## Audit Report Example

```json
{
  "contract": "VotingContract",
  "score": 80,
  "passed": true,
  "summary": {
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 4,
    "info": 0
  },
  "findings": [
    {
      "type": "Missing Input Validation",
      "severity": "🟢 Low",
      "location": "Function: vote",
      "line": 47,
      "description": "Function accepts parameters but lacks input validation.",
      "recommendation": "Validate all input parameters at the beginning of the function."
    }
  ]
}
```

## Automatic Fixes (Coming Soon)

Future versions will support automatic fixing of simple issues:

```bash
python -m pymon.cli audit contract MyContract --fix
```

This will automatically:
- Add input validation
- Add missing events
- Fix simple access control issues
- Optimize gas usage patterns

## Security Score Calculation

The security score starts at 100 and decreases based on findings:

- **Critical**: -25 points each
- **High**: -15 points each
- **Medium**: -10 points each
- **Low**: -5 points each
- **Info**: -2 points each

Minimum score is 0.

## Troubleshooting

### Contract Not Found
```
Error: Contract 'MyContract' not found in contracts/ directory
```
**Solution:** Ensure your contract is in the `contracts/` folder.

### Audit Fails But Contract Works
Some issues don't prevent deployment but reduce security:
- Missing events (monitoring)
- Gas inefficiencies (cost)
- Missing validation (edge cases)

### False Positives
The auditor uses heuristics and may report false positives. Review each finding carefully.

## Contributing

To improve the auditor, add new vulnerability checks in `pymon/auditor.py`:

```python
def _check_new_vulnerability(self):
    """Check for new vulnerability type."""
    # Your detection logic here
    if vulnerability_found:
        self.findings.append(AuditFinding(
            vulnerability_type=VulnerabilityType.NEW_TYPE,
            severity=Severity.HIGH,
            location=f"Function: {func_name}",
            line_number=line,
            description="Description of the issue",
            recommendation="How to fix it"
        ))
```

## Summary

The PyMon Security Auditor helps you:
- 🔍 **Find vulnerabilities** before deployment
- 💰 **Save gas** with optimization suggestions
- 📚 **Learn best practices** with detailed recommendations
- ✅ **Deploy with confidence** knowing your contract is secure

Always run `python -m pymon.cli audit contract <name>` before deploying to production!
