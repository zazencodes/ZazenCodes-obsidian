---
date: 2022-01-14
tags:
    - doc
hubs:
    - "[[solidity]]"
urls:
    - https://consensys.github.io/smart-contract-best-practices/
---

# Consensys Solidity Best Practices

Avoid reentancy attack:

    1. Do pre-check validations (if any)
    2. Update state
    3. Perform action (e.g. mint, transfer)


- Avoid externam calls or at least flag calls to untrusted contracts

- do not use transfer() or send()

```
// bad
contract Vulnerable {
    function withdraw(uint256 amount) external {
        // This forwards 2300 gas, which may not be enough if the recipient
        // is a contract and gas costs change.
        msg.sender.transfer(amount);
    }
}

// good
contract Fixed {
    function withdraw(uint256 amount) external {
        // This forwards all available gas. Be sure to check the return value!
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success, "Transfer failed.");
    }
}
```


