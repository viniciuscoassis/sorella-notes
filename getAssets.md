# Análise da Função getAssets do Angstrom
### Função Original
```js
function getAssets(Pair self, bool zeroToOne) internal pure returns (address assetIn, address assetOut) {
    assembly ("memory-safe") {
        let offsetIfZeroToOne := shl(5, zeroToOne)  // equivalent to: zeroToOne ? 32 : 0
        assetIn := mload(add(self, xor(offsetIfZeroToOne, 0x20)))
        assetOut := mload(add(self, offsetIfZeroToOne))
    }
}
```
### Função Equivalente (Sem Assembly)
```js
function getAssets(Pair self, bool zeroToOne) internal pure returns (address assetIn, address assetOut) {
    address asset0 = self.getAsset0();  // localizado em offset 0x00
    address asset1 = self.getAsset1();  // localizado em offset 0x20
    
    if (zeroToOne) {
        assetIn = asset0;
        assetOut = asset1;
    } else {
        assetIn = asset1;
        assetOut = asset0;
    }
}
```

## Análise de Bits
### 1. Shift Left (shl)
```
shl(5, zeroToOne)
- Se zeroToOne = 1:
  00000001 -> 00100000 (0x20 ou 32 em decimal)
- Se zeroToOne = 0:
  00000000 -> 00000000 (0x00 ou 0 em decimal)
```
### 2. XOR Operation
O XOR (^) retorna 1 apenas quando os bits são diferentes:
```
Caso zeroToOne = true:
  offsetIfZeroToOne = 0x20
  xor(0x20, 0x20) = 0x00
  00100000
  00100000
  --------
  00000000

Caso zeroToOne = false:
  offsetIfZeroToOne = 0x00
  xor(0x00, 0x20) = 0x20
  00000000
  00100000
  --------
  00100000
```
