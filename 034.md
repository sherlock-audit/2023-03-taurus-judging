cducrest-brainbot

high

# initializer modifiers should be onlyInitializing

## Summary

In multiple places throughout the code, modifiers `initializer` are used for "init" functions that are called by parent functions. These functions should have the `onlyInitializing` modifier instead.

These modifiers come from `@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol`.

## Vulnerability Detail

The `initializer` modifier fails when the contract is initializing or initialized unless the contract is under construction (called within a constructor). It sets the `_initialized` value to true (1) and `_initializing` to true before execution and resets `_initializing` to false after execution.

The `onlyInitializing` modifier requires that the call is done when `_initializing` is true.

The logic is that only the single topmost function should have the `initializer` modifier, other functions in the call chain should have the `onlyInitializing` modifier.

If a function with the `initializer` modifier calls another function with the `initializer` outside of the constructor, the call will revert.

## Impact

Init calls will revert in functions calling a wrongly modified child init functions:
- `initialize` of `GmxYieldAdapter`
- `__BaseVault_init` of `BaseVault` (abstract)

## Code Snippet

initializer modifier:

```solidity
    modifier initializer() {
        bool isTopLevelCall = !_initializing;
        require(
            (isTopLevelCall && _initialized < 1) || (!AddressUpgradeable.isContract(address(this)) && _initialized == 1),
            "Initializable: contract is already initialized"
        );
        _initialized = 1;
        if (isTopLevelCall) {
            _initializing = true;
        }
        _;
        if (isTopLevelCall) {
            _initializing = false;
            emit Initialized(1);
        }
    }
```

onlyInitializing modifier: 

```solidity
    modifier onlyInitializing() {
        require(_initializing, "Initializable: contract is not initializing");
        _;
    }
```

Init functions with `initializer` instead of `onlyInitializing`:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/BaseVault.sol#L68

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Controller/ControllableUpgradeable.sol#L19

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/TauDripFeed.sol#L32

Function with correct `initializer` modifier:

https://github.com/sherlock-audit/2023-03-taurus/blob/main/taurus-contracts/contracts/Vault/YieldAdapters/GMX/GmxYieldAdapter.sol#L16

## Tool used

Manual Review

## Recommendation

Replace the modifier from `initializer` to `onlyInitializing`. Add necessary `initializer` function (e.g. if you intend to deploy a `ControllableUpgradeable` contract with no parents, it'll need both an `initializer` and `onlyInitializing` function).