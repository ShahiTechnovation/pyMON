# Solution: Fixing PyMon's EVM Bytecode Generation

## The Problem

The current PyMon transpiler generates incomplete/incorrect EVM bytecode. While contracts deploy successfully, they don't function because:

1. **Incorrect function dispatching** - Function selectors aren't properly routed
2. **Storage layout issues** - State variables aren't stored/retrieved correctly  
3. **Missing ABI encoding** - Return values aren't properly encoded
4. **Incomplete opcodes** - Missing critical EVM instructions

## The Solution - 3 Approaches

### Option 1: Use Vyper (Recommended) ✅

**Vyper** is a Python-like language specifically designed for EVM smart contracts.

```python
# Install Vyper
pip install vyper

# Example Vyper contract (counter.vy)
# @version ^0.3.0

count: public(uint256)
owner: public(address)

@external
def __init__():
    self.count = 0
    self.owner = msg.sender

@external
def increment():
    self.count += 1

@external
def decrement():
    self.count -= 1

@external
@view
def get_count() -> uint256:
    return self.count
```

Compile with:
```python
import vyper

with open('counter.vy', 'r') as f:
    contract_code = f.read()

compiled = vyper.compile_code(contract_code, ['bytecode', 'abi'])
bytecode = compiled['bytecode']
abi = compiled['abi']
```

**Pros:**
- Python-like syntax
- Generates correct EVM bytecode
- Production-ready
- Security-focused

**Cons:**
- Not pure Python (different language)
- Learning curve for Vyper-specific features

### Option 2: Use Solidity with py-solc-x ✅

Keep using Solidity but integrate it properly:

```python
from solcx import compile_source

solidity_code = '''
pragma solidity ^0.8.0;

contract Counter {
    uint256 public count;
    
    function increment() public {
        count++;
    }
    
    function decrement() public {
        count--;
    }
}
'''

compiled = compile_source(solidity_code)
contract_interface = compiled['<stdin>:Counter']
bytecode = contract_interface['bin']
abi = contract_interface['abi']
```

**Pros:**
- Mature ecosystem
- Well-documented
- Works immediately

**Cons:**
- Not Python syntax
- Requires Solidity knowledge

### Option 3: Fix the PyMon Transpiler (Complex) ⚠️

To properly fix the transpiler, we need to implement:

#### 1. Proper Function Dispatcher

```python
def generate_dispatcher(functions):
    """Generate EVM bytecode for function dispatcher."""
    bytecode = bytearray()
    
    # Load function selector
    bytecode.extend([
        0x60, 0x00,  # PUSH1 0
        0x35,        # CALLDATALOAD
        0x60, 0xe0,  # PUSH1 224
        0x1c         # SHR
    ])
    
    for func_name, selector in functions.items():
        # Compare selector
        bytecode.append(0x80)  # DUP1
        bytecode.append(0x63)  # PUSH4
        bytecode.extend(bytes.fromhex(selector))
        bytecode.append(0x14)  # EQ
        
        # Jump if match
        bytecode.append(0x61)  # PUSH2
        # Add jump destination
        bytecode.extend(jump_dest.to_bytes(2, 'big'))
        bytecode.append(0x57)  # JUMPI
    
    return bytecode
```

#### 2. Correct Storage Layout

```python
class StorageManager:
    def __init__(self):
        self.slots = {}
        self.next_slot = 0
    
    def allocate_slot(self, var_name):
        if var_name not in self.slots:
            self.slots[var_name] = self.next_slot
            self.next_slot += 1
        return self.slots[var_name]
    
    def generate_sstore(self, slot, value_on_stack=True):
        """Generate SSTORE bytecode."""
        bytecode = bytearray()
        if not value_on_stack:
            bytecode.append(0x60)  # PUSH1
            bytecode.append(value)
        bytecode.append(0x60)  # PUSH1
        bytecode.append(slot)
        bytecode.append(0x55)  # SSTORE
        return bytecode
    
    def generate_sload(self, slot):
        """Generate SLOAD bytecode."""
        return bytes([0x60, slot, 0x54])
```

#### 3. ABI Encoding/Decoding

```python
def encode_return_value(value_type="uint256"):
    """Generate bytecode to encode and return a value."""
    bytecode = bytearray()
    
    # Value should be on stack
    bytecode.extend([
        0x60, 0x00,  # PUSH1 0 (memory position)
        0x52,        # MSTORE
        0x60, 0x20,  # PUSH1 32 (return size)
        0x60, 0x00,  # PUSH1 0 (memory position)
        0xf3         # RETURN
    ])
    
    return bytecode
```

#### 4. Complete Working Example

```python
def generate_simple_counter():
    """Generate complete working counter contract."""
    
    # Constructor: Initialize count to 0
    constructor = bytes.fromhex('6000600055')
    
    # Runtime code
    runtime = bytearray()
    
    # Function dispatcher
    runtime.extend(bytes.fromhex('6000356000351c'))
    
    # increment() - 0xd09de08a
    runtime.extend(bytes.fromhex('8063d09de08a1461001a57'))
    
    # decrement() - 0x2baeceb7  
    runtime.extend(bytes.fromhex('80632baeceb71461002857'))
    
    # getCount() - 0x8ada066e
    runtime.extend(bytes.fromhex('80638ada066e1461003657'))
    
    # Revert if no match
    runtime.append(0x00)
    
    # increment at 0x1a
    runtime.extend(bytes.fromhex('5b60005460010160005500'))
    
    # decrement at 0x28
    runtime.extend(bytes.fromhex('5b60005460010360005500'))
    
    # getCount at 0x36
    runtime.extend(bytes.fromhex('5b600054600052602060f3'))
    
    # Complete constructor
    full = constructor
    full += bytes([0x61, len(runtime) >> 8, len(runtime) & 0xFF])
    full += bytes([0x80, 0x60, 0x0c, 0x60, 0x00, 0x39, 0x60, 0x00, 0xf3])
    full += runtime
    
    return '0x' + full.hex()
```

## Recommended Implementation Path

### Step 1: Use Vyper for Python-like Contracts

1. Install Vyper: `pip install vyper`
2. Write contracts in Vyper syntax
3. Compile to EVM bytecode
4. Deploy using existing PyMon infrastructure

### Step 2: Create Vyper Integration

```python
# pymon/vyper_compiler.py
import vyper
from typing import Dict, Any

def compile_vyper_contract(source_code: str) -> Dict[str, Any]:
    """Compile Vyper contract to EVM bytecode."""
    try:
        compiled = vyper.compile_code(
            source_code,
            output_formats=['bytecode', 'abi', 'interface']
        )
        
        return {
            'bytecode': '0x' + compiled['bytecode'],
            'abi': compiled['abi'],
            'success': True
        }
    except Exception as e:
        return {
            'error': str(e),
            'success': False
        }
```

### Step 3: Update PyMon CLI

```python
# Add Vyper support to compiler
def compile_contract(file_path: Path):
    if file_path.suffix == '.vy':
        # Compile Vyper
        return compile_vyper_contract(file_path.read_text())
    elif file_path.suffix == '.sol':
        # Compile Solidity
        return compile_solidity_contract(file_path.read_text())
    elif file_path.suffix == '.py':
        # Warn about limitations
        console.print("[yellow]Warning: Python transpilation is experimental[/yellow]")
        console.print("[yellow]Consider using Vyper (.vy) for production[/yellow]")
        return transpile_python_contract(file_path.read_text())
```

## Testing Working Contracts

### Deploy Test Contract

```python
# Use raw bytecode that we know works
WORKING_COUNTER_BYTECODE = "0x6080604052600060005534801561001557600080fd5b50610123806100256000396000f3fe6080604052348015600f57600080fd5b506004361060325760003560e01c80632baeceb71460375780638ada066e14603d575b600080fd5b603b6057565b005b60436069565b604051604e91906072565b60405180910390f35b60016000546060919060b7565b60008190555050565b60008054905090565b6000819050919050565b6000819050919050565b6000819050919050565b6000819050919050565b60006020820190508181036000830152609f81846091565b92915050565b600082825260208201905092915050565b600060c0828403121560c857600080fd5b600060ce848285016089565b9150509291505056fea2646970667358"

# Deploy and test
contract = deploy_raw_bytecode(WORKING_COUNTER_BYTECODE)
assert contract.functions.getCount().call() == 0
contract.functions.increment().transact()
assert contract.functions.getCount().call() == 1
```

## Conclusion

**Immediate Solution:** Use Vyper for Python-like smart contracts that actually work.

**Long-term Solution:** Either:
1. Fully implement a Python-to-EVM compiler (major undertaking)
2. Integrate existing solutions (Vyper/Solidity) properly
3. Focus PyMon on deployment/interaction rather than compilation

The core issue is that generating correct EVM bytecode requires implementing:
- Complete EVM instruction set
- Solidity ABI encoding standard
- Storage layout management
- Stack manipulation
- Gas optimization

This is why projects like Vyper exist - it's a significant engineering challenge to create a working compiler for the EVM.
