QA1. More topics need to be added to the emit.

a) Timestamp needs to be added to show when the fee was changed.
 [https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L544](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L544)
```
emit ChangedFees(fees, proposedFees);
```

b) Timestamp needs to be added to show when the recipient was changed.
[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L556](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L556)
```
emit FeeRecipientUpdated(feeRecipient, _feeRecipient);
```

c) Timestamp needs to be added to show when ragequit is changed. 

[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L635](https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L635)
```
emit QuitPeriodSet(quitPeriod);
```

d) Timestamp needs to be added to show when adaptor is changed. 
[https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L606(https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/Vault.sol#L606)
```
emit ChangedAdapter(adapter, proposedAdapter);
```
and more for the need of timestamp as a topic:
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L755

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L795

https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L840

Needs to emit template category
https://github.com/code-423n4/2023-01-popcorn/blob/d95fc31449c260901811196d617366d6352258cd/src/vault/VaultController.sol#L868

QA2. Use safeApprove and safeTransfer and safeTransferFrom instead of calling directly the ERC20-standard counterparts. The safe versions use lower level calls to handle both cases from the ERC20 implementation: their functions might or might not return a value.  The non-safe version does not handle both cases and ignores return values, which can leave failure undetected. 

[https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol)
