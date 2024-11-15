## EncodingTypes
### ToB orders
- 1 byte variant map (flags for useInternal, zeroForOne, recipient presence, and signature type)
- 16 bytes quantityIn
- 16 bytes quantityOut
- 16 bytes maxGasAsset0
- 16 bytes gasUsedAsset0
- 2 bytes pairIndex
- Optional recipient address (20 bytes if present)
- Signature data

### User orders
- 1 byte variant map
- 4 bytes refId
- 2 bytes pairIndex
- 32 bytes minPrice
- Optional recipient (20 bytes)
- Optional hook data
- Order validation data
- Amounts and fees
- Signature data

### Pair
- 2 bytes indexA
- 2 bytes indexB
- 2 bytes storeIndex
- 32 bytes price data

Each section typically starts with a length prefix (3 bytes) followed by the actual encoded data.
Note that to ensure pair uniqueness `.index0` **must** be less than `.index1`.

[length (3 bytes)][pair1][pair2]...[pairN]

[
  Assets encoding
  Pairs encoding
  Pool updates encoding
  ToB orders encoding
  User orders encoding
]
### Assets Encoding
- addr (20 bytes): Token address
- save (16 bytes): Amount to save as network fee
- take (16 bytes): Amount to take from Uniswap
- settle (16 bytes): Amount to repay to Uniswap

[length (3 bytes)][asset1][asset2]...[assetN]
The assets must be sorted by address.

### Pool Updates
- Variant byte (1 byte): Contains flags for zeroForOne and onlyCurrent
- Pair index (2 bytes): Index into pairs array
- Amount in (16 bytes): Input amount
- **Rewards update encoding**

The rewards update encoding varies based on whether it's onlyCurrent
##### If onlyCurrent:
- Just the current quantity (16 bytes)
##### Otherwise:
- Start tick (3 bytes)
- Start liquidity (16 bytes)
- Encoded quantities array with length prefix
### Variant Map
- QTY_PARTIAL_BIT: Whether the order is partial (vs exact)
- IS_STANDING_BIT: Whether it's a standing order (vs flash)
- IS_EXACT_IN_BIT: Whether exact input (vs exact output)
- HAS_HOOK_BIT: Whether there's a hook
- USE_INTERNAL_BIT: Whether to use internal liquidity
- HAS_RECIPIENT_BIT: Whether there's a recipient
- IS_ECDSA_BIT: Whether using ECDSA signatures
- ZERO_FOR_ONE_BIT: Direction of the swap
