| Total Low issues |
|------------------|

| Risk   | Issues Details                                                             | Number        |
|--------|----------------------------------------------------------------------------|---------------|
| [L-01] | No Storage Gap for Upgradeable contracts                                   | 4             |
| [L-02] | Loss of precision due to rounding                                          | 2             |
| [L-03] | Allows malleable `SECP256K1` signatures                                    | 3             |
| [L-04] | Integer overflow by unsafe casting                                         | 17            |
| [L-05] | Misleading NatSpec                                                         | 1             |
| [L-06] | Value is not validated to be different than the existing one               | 5             |
| [L-07] | Unhandled return values of `transfer` and `transferFrom`                   | 3             |
| [L-08] | Not all tokens support approve-max                                         | 8             |
| [L-09] | ERC4626 does not work with fee-on-transfer tokens                          | 2             |
| [L-10] | ERC4626 implmentation is not fully up to EIP-4626's specification          | 2             |
| [L-11] | Missing checks for `address(0)`                                            | 2             |
| [L-12] | Missing Event for initialize                                               | 4             |

| Total Non-Critical issues |
|---------------------------|

| Risk    | Issues Details                                                                             | Number        |
|---------|--------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Open TODO                                                                                  | 1             |
| [NC-02] | Include `@return` parameters in NatSpec comments                                           | All Contracts |
| [NC-03] | Lines are too long                                                                         | 7             |
| [NC-04] | Signature Malleability of EVM's `ecrecover()`                                              | 3             |
| [NC-05] | Use scientific notation rather than exponentiation                                         | 3             |
| [NC-06] | Use `bytes.concat()` and `string.concat()`                                                 | 7             |
| [NC-07] | Use a more recent version of solidity                                                      | All Contracts |
| [NC-08] | Lock pragmas to specific compiler version                                                  | All Contracts |
| [NC-09] | Constants in comparisons should appear on the left side                                    | 15            |
| [NC-10] | Function writing does not comply with the `Solidity Style Guide`                           | All Contracts |
| [NC-11] | NatSpec comments should be increased in contracts                                          | All Contracts |
| [NC-12] | Follow Solidity standard naming conventions                                                | All Contracts |
| [NC-13] | Consider using `delete` rather than assigning zero to clear values                         | 2             |
| [NC-14] | Critical changes should use-two step procedure                                             | 1             |
| [NC-15] | Contracts should have full test coverage                                                   | All Contracts |
| [NC-16] | Using vulnerable dependency of solmate                                                     | 1             |
| [NC-17] | Use SMTChecker                                                                             |               |
| [NC-18] | Not using the named return variables anywhere in the function is confusing                 | 1             |
| [NC-19] | Using vulnerable dependency of OpenZeppelin                                                | 1             |
| [NC-20] | Events that mark critical parameter changes should contain both the old and the new value  | 1             |
| [NC-21] | Add a timelock to critical functions                                                       | 7             |
| [NC-22] | Need Fuzzing test                                                                          | All Contracts |
| [NC-23] | Add NatSpec comment to mapping                                                             | 9             |

## [L-01] No Storage Gap for Upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) for a description of this storage variable. 

#### Lines of code 

- [BeefyAdapter.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol)
- [MultiRewardStaking.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol)
- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)
- [YearnAdapter.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol)

#### Recommended Mitigation Steps

```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-02] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

```solidity
                lockedProfit -
                ((lockedFundsRatio * lockedProfit) / DEGRADATION_COEFFICIENT);
```

#### Lines of code 

- [MultiRewardEscrow.sol#L181](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L181)
- [YearnAdapter.sol#L122](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L122)

## [L-03] Allows malleable `SECP256K1` signatures

#### Description

Homestead [(EIP-2)](https://eips.ethereum.org/EIPS/eip-2) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.

> Ref: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149

#### Lines of code 

- [AdapterBase.sol#L646](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646)
- [MultiRewardStaking.sol#L459](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L459)
- [Vault.sol#L678](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L678)

#### Recommended Mitigation Steps

Use OpenZeppelin's [`ECDSA`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol) library for signature verification.

## [L-04] Integer overflow by unsafe casting

#### Description

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

```solidity
      Math.min(
        (escrow.balance * (block.timestamp - uint256(escrow.lastUpdateTime))) /
          (uint256(escrow.end) - uint256(escrow.lastUpdateTime)),
        escrow.balance
      );
```

#### Lines of code 

- [MultiRewardEscrow.sol#L181](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L181)
- [MultiRewardEscrow.sol#L182](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L182)
- [MultiRewardStaking.sol#L197](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L197)
- [MultiRewardStaking.sol#L357](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L357)
- [MultiRewardStaking.sol#L359](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L359)
- [MultiRewardStaking.sol#L398](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L398)
- [MultiRewardStaking.sol#L406](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L406)
- [MultiRewardStaking.sol#L427](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L427)
- [Vault.sol#L144](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L144)
- [Vault.sol#L179](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L179)
- [Vault.sol#L220](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L220)
- [Vault.sol#L264](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L264)
- [Vault.sol#L330](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L330)
- [Vault.sol#L341](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L341)
- [Vault.sol#L363](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L363)
- [Vault.sol#L388](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L388)

#### Recommended Mitigation Steps

Use OpenZeppelin [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) library.

## [L-05] Misleading NatSpec

#### Description

This line is missliding because the function should look at `canCreate()` and not the `onlyOwner()`:

```solidity
  /**
   * @notice Deploy a new staking contract. Caller must be owner.
   * @param asset The staking token for the new contract.
   * @dev Deploys `MultiRewardsStaking` based on the latest templateTemplateKey.
   */
  function deployStaking(IERC20 asset) external canCreate returns (address) {
    _verifyToken(address(asset));
    return _deployStaking(asset, deploymentController);
  }
```

#### Lines of code 

- [VaultController.sol#L274](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L274)

#### Recommended Mitigation Steps

Fix the NtaSpec comment:

```solidity
  /**
   * @notice Deploy a new staking contract. The caller must be added as an allowed creator.
   * @param asset The staking token for the new contract.
   * @dev Deploys `MultiRewardsStaking` based on the latest templateTemplateKey.
   */
  function deployStaking(IERC20 asset) external canCreate returns (address) {
    _verifyToken(address(asset));
    return _deployStaking(asset, deploymentController);
  }
```

## [L-06] Value is not validated to be different than the existing one

#### Description

While the value is validated to be in the constant bounds, it is not validated to be different than the existing one. Queueing the same value will cause multiple abnormal events to be emitted, will ultimately result in a no-op situation.

```solidity
  function setPerformanceFee(uint256 newFee) external onlyOwner {
    // Dont take more than 20% performanceFee
    if (newFee > 2e17) revert InvalidPerformanceFee(newFee);

    emit PerformanceFeeChanged(performanceFee, newFee);

    performanceFee = newFee;
  }
```

#### Lines of code 

- [AdapterBase.sol#L500](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L500)
- [AdapterBase.sol#L549](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549)
- [Vault.sol#L629](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L629)
- [VaultController.sol#L751](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L751)
- [VaultController.sol#L791](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L791)

#### Recommended Mitigation Steps

```solidity
  function setPerformanceFee(uint256 newFee) external onlyOwner {
    if (newFee == performanceFee) revert();
    // Dont take more than 20% performanceFee
    if (newFee > 2e17) revert InvalidPerformanceFee(newFee);

    emit PerformanceFeeChanged(performanceFee, newFee);

    performanceFee = newFee;
  }
```

## [L-07] Unhandled return values of `transfer` and `transferFrom`

#### Description

ERC20 implementations are not always consistent. Some implementations of ```transfer``` and ```transferFrom``` could return false on failure instead of reverting. It is safer to check the return value and revert on false to prevent these failures.

This is a known issue with solmate and Opanzeppelin ERC20 libraries, For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens, hence Tether (USDT) `transfer()` and `transferFrom()` functions do not return booleans as the specification requires, and instead have no return value.

#### Lines of code 

- [MultiRewardStaking.sol#L182](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L182)
- [VaultController.sol#L457](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L457)
- [VaultController.sol#L526](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L526)

#### Recommended Mitigation Steps

Check the return value and revert on false.

## [L-08] Not all tokens support approve-max

#### Description

Some tokens consider uint256 as just another value. If one of these tokens is used and the approval balance eventually reaches only a couple wei, nobody will be able to send funds because there won't be enough approval.

```solidity
      rewardToken.safeApprove(address(escrow), type(uint256).max);
```

#### Lines of code 

- [MultiRewardStaking.sol#L271](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L271)
- [BeefyAdapter.sol#L80](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L80)
- [BeefyAdapter.sol#L83](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L83)
- [Vault.sol#L80](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L80)
- [Vault.sol#L610](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L610)
- [VaultController.sol#L452](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L452)
- [VaultController.sol#L456](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L456)
- [YearnAdapter.sol#L54](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L54)

## [L-09] ERC4626 does not work with fee-on-transfer tokens

#### Description

The ERC4626 [deposit](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L110-L122)/[mint](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L129-L141) functions do not work well with fee-on-transfer tokens as the `assets` variable is the pre-fee amount, including the fee, whereas the `totalAssets` do not include the fee anymore.

This can be abused to mint more shares than desired.

#### Lines of code 

- [AdapterBase.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol)
- [Vault.sol](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol)

#### Recommended Mitigation Steps

`assets` should be the amount excluding the fee (i.e the amount the contract actually received), therefore it's recommended to use the balance change before and after the transfer instead of the amount.

## [L-10] ERC4626 implmentation is not fully up to EIP-4626's specification

#### Description

Must return the maximum amount of shares mint would allow to be deposited to receiver and not cause a revert, which must not be higher than the actual maximum that would be accepted (it should underestimate if necessary).

This assumes that the user has infinite assets, i.e. must not rely on balanceOf of asset.

```solidity
    function maxDeposit(address)
        public
        view
        virtual
        override
        returns (uint256)
    {
        return paused() ? 0 : type(uint256).max;
    }
```

```solidity
    function maxMint(address) public view virtual override returns (uint256) {
        return paused() ? 0 : type(uint256).max;
    }

```

#### Lines of code 

- [AdapterBase.sol#L404-L412](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L404-L412)
- [AdapterBase.sol#L419-L421](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L419-L421)

#### Recommended Mitigation Steps

`maxMint()` and `maxDeposit()` should reflect the limitation of maxSupply.

## [L-11] Missing checks for `address(0)`

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

```solidity
  constructor(
    address _owner,
    ICloneFactory _cloneFactory,
    ICloneRegistry _cloneRegistry,
    ITemplateRegistry _templateRegistry
  ) Owned(_owner) {
    cloneFactory = _cloneFactory;
    cloneRegistry = _cloneRegistry;
    templateRegistry = _templateRegistry;
  }
```

#### Lines of code 

- [VaultController.sol#L53-L70](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L53-L70)
- [DeploymentController.sol#L35-L44](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/DeploymentController.sol#L35-L44)

#### Recommended Mitigation Steps

Add checks for `address(0)` when assigning values to address state variables.	

## [L-12] Missing Event for initialize

#### Description

Events help non-contract tools to track changes, and events prevent users from being surprised by changes Issuing event-emit during initialization is a detail that many projects skip.

```solidity
function initialize(
        bytes memory adapterInitData,
        address registry,
        bytes memory beefyInitData
    ) external initializer {
        (address _beefyVault, address _beefyBooster) = abi.decode(
            beefyInitData,
            (address, address)
        );
        __AdapterBase_init(adapterInitData);

        if (!IPermissionRegistry(registry).endorsed(_beefyVault))
            revert NotEndorsed(_beefyVault);
        if (IBeefyVault(_beefyVault).want() != asset())
            revert InvalidBeefyVault(_beefyVault);
        if (
            _beefyBooster != address(0) &&
            IBeefyBooster(_beefyBooster).stakedToken() != _beefyVault
        ) revert InvalidBeefyBooster(_beefyBooster);

        _name = string.concat(
            "Popcorn Beefy",
            IERC20Metadata(asset()).name(),
            " Adapter"
        );
        _symbol = string.concat("popB-", IERC20Metadata(asset()).symbol());

        beefyVault = IBeefyVault(_beefyVault);
        beefyBooster = IBeefyBooster(_beefyBooster);

        beefyBalanceCheck = IBeefyBalanceCheck(
            _beefyBooster == address(0) ? _beefyVault : _beefyBooster
        );

        IERC20(asset()).approve(_beefyVault, type(uint256).max);

        if (_beefyBooster != address(0))
            IERC20(_beefyVault).approve(_beefyBooster, type(uint256).max);
    }
```

#### Lines of code 

- [YearnAdapter.sol#L34-L55](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L34-L55)
- [Vault.sol#L57-L98](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L57-L98)
- [MultiRewardStaking.sol#L41-L57](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L41-L57)
- [BeefyAdapter.sol#L46-L84](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/beefy/BeefyAdapter.sol#L46-L84)

#### Recommended Mitigation Steps

Add Event-Emit

## [NC-01] Open TODO

#### Description

The code that contains "open todos" reflects that the development is not finished and that the code can change a posteriori, prior release, with or without audit.

```solidity
    // TODO use deterministic fee recipient proxy
```

#### Lines of code 

- [AdapterBase.sol#L516](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L516)

## [NC-02] Include `@return` parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`. Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Include the `@return` argument in the NatSpec comments.

## [NC-03] Lines are too long

#### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

> Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Lines of code 

- [YearnAdapter.sol#L6](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L6)
- [MultiRewardEscrow.sol#L49](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L49)
- [MultiRewardStaking.sol#L7](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L7)
- [MultiRewardStaking.sol#L22](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L22)
- [MultiRewardStaking.sol#L380](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L380)
- [VaultController.sol#L477](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L477)
- [AdapterBase.sol#L6](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L6)

## [NC-04] Signature Malleability of EVM's `ecrecover()`

#### Description

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin's ECDSA library.

#### Lines of code 

- [AdapterBase.sol#L646](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L646)
- [MultiRewardStaking.sol#L459](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L459)
- [Vault.sol#L678](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L678)

#### Recommended Mitigation Steps

Use the ecrecover function from OpenZeppelin's ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

## [NC-05] Use scientific notation rather than exponentiation

#### Description

While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist.

```solidity
    uint256 constant DEGRADATION_COEFFICIENT = 10**18;
```

#### Lines of code 

- [YearnAdapter.sol#L25](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L25)
- [MultiRewardStaking.sol#L274](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L274)
- [MultiRewardStaking.sol#L406](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L406)

#### Recommended Mitigation Steps

Use scientific notation `(e.g. 1e18)` rather than exponentiation `(e.g. 10**18)`.

## [NC-06] Use `bytes.concat()` and `string.concat()`

#### Description

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

 > https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

- [AdapterBase.sol#L648](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L648)
- [MultiRewardEscrow.sol#L10](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L104)
- [MultiRewardStaking.sol#L49](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L49)
- [MultiRewardStaking.sol#L50](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L50)
- [MultiRewardStaking.sol#L461](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L461)
- [Vault.sol#L94](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L94)
- [Vault.sol#L680](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L680)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()`.

## [NC-07] 	Use a more recent version of solidity

#### Description

For security, it is best practice to use the [latest Solidity version](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

```solidity
pragma solidity ^0.8.15;
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Old version of Solidity is used `(0.8.15)`, newer version can be used `(0.8.17)`.

## [NC-08] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

> Ref: https://swcregistry.io/docs/SWC-103

```solidity
pragma solidity ^0.8.15;
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/):  Lock pragmas to specific compiler version.

## [NC-09] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
    if (amount == 0) revert ZeroAmount();
```

#### Lines of code 

- [YearnAdapter.sol#L90](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/yearn/YearnAdapter.sol#L90)
- [MultiRewardEscrow.sol#L97](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L97)
- [MultiRewardEscrow.sol#L98](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L98)
- [MultiRewardEscrow.sol#L160](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L160)
- [MultiRewardEscrow.sol#L172](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L172)
- [MultiRewardEscrow.sol#L173](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L173)
- [MultiRewardEscrow.sol#L174](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L174)
- [MultiRewardStaking.sol#L174](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L174)
- [MultiRewardStaking.sol#L258](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L258)
- [MultiRewardStaking.sol#L299](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L299)
- [MultiRewardStaking.sol#L300](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L300)
- [MultiRewardStaking.sol#L301](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L301)
- [MultiRewardStaking.sol#L325](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L325)
- [MultiRewardStaking.sol#L331](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L331)
- [MultiRewardStaking.sol#L420](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L420)

#### Recommended Mitigation Steps

```solidity
    if (0 == amount) revert ZeroAmount();
```

## [NC-10] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external` / `public` / `internal` / `private`
- `view` / `pure`

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Follow [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html?highlight=Style#style-guide).

## [NC-11] NatSpec comments should be increased in contracts

#### Description

It is recommended that Solidity contracts are fully annotated using NatSpec, it is clearly stated in the Solidity official documentation.

- In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

- Some code analysis programs do analysis by reading NatSpec details, if they can't see the tags `(@param, @dev, @return)`, they do incomplete analysis.


#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Include [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html#natspec-format) comments in the codebase.

## [NC-12] Follow Solidity standard naming conventions 

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Follow solidity standard [naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).

## [NC-13] Consider using `delete` rather than assigning zero to clear values

#### Description

The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

```solidity
      accruedRewards[user][_rewardTokens[i]] = 0;
```

#### Lines of code 

- [AdapterBase.sol#L577](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L577)
- [MultiRewardStaking.sol#L186](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L186)

#### Recommended Mitigation Steps

```solidity
      delete accruedRewards[user][_rewardTokens[i]];
```

## [NC-14] Critical changes should use-two step procedure

#### Description

Critical changes should use-two step procedure:

```solidity
    /**
     * @notice Change `feeRecipient`. Caller must be Owner.
     * @param _feeRecipient The new fee recipient.
     * @dev Accrued fees wont be transferred to the new feeRecipient.
     */
    function setFeeRecipient(address _feeRecipient) external onlyOwner {
        if (_feeRecipient == address(0)) revert InvalidFeeRecipient();

        emit FeeRecipientUpdated(feeRecipient, _feeRecipient);

        feeRecipient = _feeRecipient;
    }

```

#### Lines of code 

- [Vault.sol#L548-L559](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L548-L559)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a `pendingFeeRecipient` and the new address should then claim its role.

## [NC-15] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

```java script
- What is the overall line coverage percentage provided by your tests?: 94.52%
```

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [NC-16] Using vulnerable dependency of solmate

#### Description

The `package.json` configuration file says that the project is using `6.6.2` of solmate which has a not last update version.

```java script
  "name": "solmate",
  "license": "AGPL-3.0-only",
  "version": "6.6.2",
  "description": "Modern, opinionated and gas optimized building blocks for smart contract development.",
  "files": [
    "src/**/*.sol"
  ],
```

#### Lines of code 

- [package.json#L4](https://github.com/transmissions11/solmate/blob/564e9f1606c699296420500547c47685818bcccf/package.json#L4)

#### Recommended Mitigation Steps

Use solmate latest version `6.7.0`.

## [NC-17] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Ref: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

#### Recommended Mitigation Steps

Use SMTChecker.

## [NC-18] Not using the named return variables anywhere in the function is confusing

#### Description

Not using the named return variables anywhere in the function is confusing: 

```solidity
    function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256 shares)
    {
        uint256 _totalSupply = totalSupply();
        uint256 _totalAssets = totalAssets();
        return
            (assets == 0 || _totalSupply == 0 || _totalAssets == 0)
                ? assets
                : assets.mulDiv(_totalSupply, _totalAssets, rounding);
    }
```

#### Lines of code 

- [AdapterBase.sol#L380](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L380)

#### Recommended Mitigation Steps

Consider changing the variable to be an unnamed one:

```solidity
    function _convertToShares(uint256 assets, Math.Rounding rounding)
        internal
        view
        virtual
        override
        returns (uint256)
    {
        uint256 _totalSupply = totalSupply();
        uint256 _totalAssets = totalAssets();
        return
            (assets == 0 || _totalSupply == 0 || _totalAssets == 0)
                ? assets
                : assets.mulDiv(_totalSupply, _totalAssets, rounding);
    }
```

## [NC-19] Using vulnerable dependency of OpenZeppelin

#### Description

The package.json configuration file says that the project is using 4.7.1 of OpenZeppelin which has a not last update version.

```solidity
{
  "name": "@openzeppelin/contracts-upgradeable",
  "description": "Secure Smart Contract library for Solidity",
  "version": "4.7.1",
  "files": [
    "**/*.sol",
    "/build/contracts/*.json",
    "!/mocks/**/*"
  ],
```

#### Recommended Mitigation Steps

Use patched versions Latest non vulnerable version 4.8.0.

## [NC-20] Events that mark critical parameter changes should contain both the old and the new value

#### Description

Events that mark critical parameter changes should contain both the old and the new value:

```solidity
    function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
        if (_quitPeriod < 1 days || _quitPeriod > 7 days)
            revert InvalidQuitPeriod();

        quitPeriod = _quitPeriod;

        emit QuitPeriodSet(quitPeriod);
    }
```

#### Lines of code 

- [Vault.sol#L629-L636](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L629-L636)

#### Recommended Mitigation Steps

```solidity
    
    event QuitPeriodSet(uint256 previousQuitPeriod, uint256 newQuitPeriod);
      
    function setQuitPeriod(uint256 _quitPeriod) external onlyOwner {
        if (_quitPeriod < 1 days || _quitPeriod > 7 days)
            revert InvalidQuitPeriod();
        
        emit QuitPeriodSet(quitPeriod, _quitPeriod);
        quitPeriod = _quitPeriod;
    }
```

## [NC-21] Add a timelock to critical functions

#### Description

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate.

#### Lines of code 

- [AdapterBase.sol#L500](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L500)
- [AdapterBase.sol#L549](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L549)
- [MultiRewardEscrow.sol#L207](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardEscrow.sol#L207)
- [Vault.sol#L629](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L629)
- [VaultController.sol#L543](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L543)
- [VaultController.sol#L751](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L751)
- [VaultController.sol#L791](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L791)

#### Recommended Mitigation Steps

Consider adding a timelock to the critical changes.

## [NC-22] Need Fuzzing test

#### Description

As Alberto Cuesta Canada said: Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

> Ref: https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-01-popcorn/tree/main/src)

#### Recommended Mitigation Step

Use fuzzing test like Echidna.

## [NC-23] Add NatSpec comment to mapping

#### Description

Add NatSpec comments describing mapping keys and values.

```solidity
    mapping(address => uint256) public nonces;
```

#### Lines of code 

- [AdapterBase.sol#L627](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/adapter/abstracts/AdapterBase.sol#L627)
- [CloneRegistry.sol#L28](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/CloneRegistry.sol#L28)
- [MultiRewardStaking.sol#L440](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/utils/MultiRewardStaking.sol#L440)
- [PermissionRegistry.sol#L26](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/PermissionRegistry.sol#L26)
- [TemplateRegistry.sol#L32](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L32)
- [TemplateRegistry.sol#L33](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L33)
- [TemplateRegistry.sol#L35](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/TemplateRegistry.sol#L35)
- [Vault.sol#L659](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/Vault.sol#L659)
- [VaultController.sol#L851](https://github.com/code-423n4/2023-01-popcorn//blob/main/src/vault/VaultController.sol#L851)

#### Recommended Mitigation Steps

```solidity
/// @dev  address(user) => uint256(nonce)
    mapping(address => uint256) public nonces;
```
