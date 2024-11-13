# Angstrom Protocol Code Walkthrough
## Overview
Angstrom is a hybrid AMM/Orderbook exchange built on Ethereum that aims to mitigate MEV for both users and Liquidity Providers. The protocol integrates with Uniswap v4 as a hook and implements its own liquidity pools with specific trading rules.

### Core Architecture
Main Contract Functionalities
1. Validating transaction batches
2. Executing and settling batches
3. Processing signed orders
4. Handling order resubmissions

### Uniswap V4 Integration
The contract functions as a Uniswap hook, building on top of Uniswap v4 with custom liquidity pools. Key characteristics:
- Custom trading rules enforced through beforeSwap hook
- No direct trading allowed with the pools

### Entry Points
The contract has two main entry points:
1. execute() - Primary entry point for batch execution
2. unlockCallback() - Uniswap v4 callback handler

### Call Context Flow
```
graph TD
    A[execute] --> B[unlock]
    B --> C[unlockCallback]
```
When performing actions with Uniswap v4 (swapping, adding liquidity), operations must be within a controlled call context to verify invariants.

### Custom Encoding Format
The protocol uses a custom encoding format for efficiency:
- Format specification: docs/pade-encoding-format.md
- Payload types: docs/payload-types.md

### Validation and Decoding Process
Key data structures decoded during validation:
```
// Asset validation
AssetArray assets;
(reader, assets) = AssetLib.readFromAndValidate(reader);

// Pair validation
PairArray pairs;
(reader, pairs) = PairLib.readFromAndValidate(reader, assets, _configStore);
```

## Execution Flow

### 1. Asset Taking (`_takeAssets`)
The settlement module handles taking tokens from Uniswap for two purposes:
- Getting swap outputs
- Flash borrowing when needed for order settlement

### 2. Pool Updates (`_updatePools`)
Handles:
- AMM swap execution
- LP reward distribution
- Custom fee computation and distribution

Key feature: Ability to reward LPs across multiple tick ranges, unlike Uniswap which only rewards current tick.

### 3. Order Processing

Two types of orders:

#### Top of Block (TOB) Orders
```markdown
- Special orders for searchers/sophisticated actors
- Used for AMM arbitrage
- Protects LPs from MEV
- Processed through off-chain auction system
- Executed first in the batch
```

#### User Orders
```markdown
- Standard limit orders
- Include price protection
- Guaranteed same price for all orders against same pair
- Note: A->B->C paths may have different pricing than direct A->C
```

### 4. Settlement (`saveAndSettle`)
Final phase that:
- Verifies all amounts are properly settled
- Processes asset list
- Validates final delta is zero
- Handles any required Uniswap paybacks

## Fee Management

Implements gas-optimized fee collection:
- Fees aren't directly accounted
- Collected in aggregated list
- Hashed and emitted as event
- Referenced in code as "Fee Summary"

## Potential Risk Areas

### 1. Pool Reward Logic
```markdown
High-risk area due to:
- Complex accumulator logic
- Edge cases with tick boundaries
- Swap crossing scenarios
```

### 2. Assembly Code
```markdown
Key areas to audit:
- Type truncation assumptions
- Assembly implementation correctness
```

### 3. Order Management
```markdown
Critical aspects:
- Order semantic correctness
- Buffer overflow prevention
- Value overwrite protection
```

### 4. Order Types Implementation
```markdown
Standing Orders:
- Uses timestamp, timeline, nonce
- Classic order structure

Flash Orders:
- Block number based
- Leverages transient storage
- Single-block execution enforcement
```
