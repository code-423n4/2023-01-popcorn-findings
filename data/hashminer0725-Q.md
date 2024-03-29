### 1. ```success``` could be defined in the first ```if``` condition
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/CloneFactory.sol#L42
```
    bool success = true;
    if (template.requiresInitData) (success, ) = clone.call(data);

    if (!success) revert DeploymentInitFailed();
```

```success``` could be defined in the first ```if``` condition. If ```template.requiresInitData``` is ```false```, checking ```!success``` is unnecessary.

### 2. Attribute values of ```fees_``` could exceed 1e18 when initializing even if the ```proposedFees``` is checked in ```proposeFees()``` function.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L88
```
    function initialize(
        IERC20 asset_,
        IERC4626 adapter_,
        VaultFees calldata fees_,
        address feeRecipient_,
        address owner
    ) external initializer {
        ...
        fees = fees_;
```
```
    function proposeFees(VaultFees calldata newFees) external onlyOwner {
        if (
            newFees.deposit >= 1e18 ||
            newFees.withdrawal >= 1e18 ||
            newFees.management >= 1e18 ||
            newFees.performance >= 1e18
        ) revert InvalidVaultFees();

        proposedFees = newFees;
```
```initialize()``` function could have a check if the attribute values of ```fees_``` are less than ```1e18```.

### 3. ```convertToShares()``` function is used twice.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L143
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L147
```
    uint256 feeShares = convertToShares(
        assets.mulDiv(uint256(fees.deposit), 1e18, Math.Rounding.Down)
    );

    shares = convertToShares(assets) - feeShares;
```
It could calculate the share amount for ```asset``` and then calculate ```feeShares``` and ```shares``` to be minted from the amount. Like doing in other functions.

### 4. ```assetsCheckpoint``` is never used.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/Vault.sol#L467
```
    uint256 public assetsCheckpoint;
```
The variable could be removed from the contract.

### 5. ```_registerCreatedVault()``` function doesn't need to have ```metadata``` parmeter.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L144
```
    function _registerCreatedVault(
      address vault,
      address staking,
      VaultMetadata memory metadata
    ) internal {
      metadata.vault = vault;
      metadata.staking = staking;
      metadata.creator = msg.sender;
```
```metadata``` is set in the function again.

### 6. Some attributes could be removed from the struct.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/interfaces/vault/IVaultRegistry.sol#L16-L21
```
    /// @notice OPTIONAL - If the asset is an Lp Token these are its underlying assets
    address[8] swapTokenAddresses;
    /// @notice OPTIONAL - If the asset is an Lp Token its the pool address
    address swapAddress;
    /// @notice OPTIONAL - If the asset is an Lp Token this is the identifier of the exchange (1 = curve)
    uint256 exchange;
```
```swapTokenAddresses```, ```swapAddress```, and ```exchange``` are never used from the contracts.

### 7. ```newCooldown``` can include ```1 day``` even if ```1 day``` is not allowed in VaultController.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L502
```
    if (newCooldown >= 1 days) revert InvalidHarvestCooldown(newCooldown);
```
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/VaultController.sol#L793
```
    if (newCooldown > 1 days) revert InvalidHarvestCooldown(newCooldown);
```

### 8. ```NoteSubmitter()``` is never used.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardStaking.sol#L226
```
    error NotSubmitter(address submitter);
```

### 9. ``` NoFee(IERC20 token)``` is never used.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/utils/MultiRewardEscrow.sol#L200
```
    error NoFee(IERC20 token);
```

### 10. ```setPermissions()``` function is only reverted if both ```newPermissions[i].endorsed``` and ```newPermissions[i].rejected``` are ```true```. But it also should be reverted if both are ```false```.
https://github.com/code-423n4/2023-01-popcorn/blob/main/src/vault/PermissionRegistry.sol#L43
```
    if (newPermissions[i].endorsed && newPermissions[i].rejected) revert Mismatch();
```

### 11. ```accrued``` could be removed from the ```Fee``` struct.
https://github.com/code-423n4/2023-01-popcorn//blob/main/src/interfaces/IMultiRewardEscrow.sol#L26
```
24    struct Fee {
25      /// @notice Accrued fee amount
26      uint256 accrued;
27      /// @notice Fee percentage in 1e18 for 100% (1 BPS = 1e14)
28      uint256 feePerc;
29    }
```
It's never used from the contracts.