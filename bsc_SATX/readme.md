## SATX Token Summary

**Contract Address on BSC:**
- **Address:** `0xfd80a436da2f4f4c42a5dbfa397064cfeb7d9508`
- **BSCScan:** [View Contract on BSCScan](https://bscscan.com/address/0xfd80a436da2f4f4c42a5dbfa397064cfeb7d9508#code)

### Overview
The SATX token contract integrates specialized logic for transfers, especially when interacting with its Uniswap V2 liquidity pair. This document breaks down its unique transfer function and associated internal checks.


### Problematic Code

#### _transfer Function [SATXtoken.sol](SATXtoken.sol#L982-L991)
```solidity
function _transfer(
    address from,
    address to,
    uint256 amount
) internal override {
    require(from != address(0), "ERC20: transfer from the zero address");
    require(to != address(0), "ERC20: transfer to the zero address");
    require(to != from, "ERC20: transfer to the same address");
    require(amount > 0);

    if(_isExcludedFromFeesVip[from] || _isExcludedFromFeesVip[to]){
        super._transfer(from, to, amount);
        return;
    }

    if(from == uniswapV2Pair){
        (bool ldxDel, bool bot, uint256 usdtAmount) = _isDelLiquidityV2();
        if(bot){
            super._transfer(from, _tokenOwner, amount);
        } else if(ldxDel){
            require(startTime.add(300) < block.timestamp, "swap not start");
            (uint256 lpDelAmount,) = getLpBalanceByUsdt(usdtAmount);
            _haveLpAmount[to] = _haveLpAmount[to].sub(lpDelAmount);
            super._transfer(from, to, amount);
        }
    }
}
```

#### _isDelLiquidityV2 Function [SATXtoken.sol](SATXtoken.sol#L1106-L1119)
```solidity
function _isDelLiquidityV2() internal view returns(bool ldxDel, bool bot, uint256 otherAmount){
    address token0 = IUniswapV2Pair(address(uniswapV2Pair)).token0();
    (uint reserves0,,) = IUniswapV2Pair(address(uniswapV2Pair)).getReserves();
    uint amount = IERC20(token0).balanceOf(address(uniswapV2Pair));
    if(token0 != address(this)){
        if(reserves0 > amount){
            otherAmount = reserves0 - amount;
            ldxDel = otherAmount >= 10**13;
        } else {
            bot = reserves0 == amount;
        }
    }
}
```

## Functional Insights

### Liquidity Check (_isDelLiquidityV2)
- **`bot` Flag:** True if the real-time balance (`amount`) matches the last snapshot reserves (`reserves0`), indicating no recent transactions have altered the liquidity state.
- **`ldxDel` Flag:** True if the difference between the reserves and the actual balance exceeds 10^13, pointing to significant liquidity movements.

### Transfer Redirects
- **Bot Condition:** If true (liquidity is stable), tokens transferred from the liquidity pool are redirected to `_tokenOwner`, not the intended recipient. 

## Exploit scenario
The `_transfer` function redirects funds to `_tokenOwner` under the bot condition, which is set when the reserves and actual balances are perfectly aligned. This condition, while indicative of no recent transactions affecting the liquidity, could be exploited by a malicious token owner or through coordinated actions that manipulate the timing of transactions to create conditions where bot is true or during normal interactions with the contract (`!`). As a result, instead of going to the *intended recipient*, the funds are transferred to the *token owner*, potentially draining the liquidity pool without typical transactional visibility.

### Reserves vs. Actual Balance
- `Reserves (reserves0)`: This is the amount of token0 (could be SATX if it is token0 in the pair) recorded as locked in the liquidity pool. This figure is updated only during transactions that impact the liquidity such as swaps, additions, or removals. It represents a snapshot in time following the last transaction that modified the pool's composition.
- `Actual Balance (amount)`: This represents the live balance of token0 currently available in the Uniswap V2 pair contract. This amount is an up-to-date reflection of the token's available liquidity that can fluctuate with each block as new transactions occur that affect the pool.

### How Liquidity Transactions Update Reserves
When liquidity is added to a Uniswap V2 pair, the following occurs:

1. Liquidity Added: A user deposits both assets of the pair (e.g., SATX and BNB on the Binance Smart Chain) into the pool.
2. Pool Tokens Minted: In return for the added liquidity, the user receives liquidity provider (LP) tokens, which represent their share of the total pool.
3. Reserves Updated: The smart contract updates the reserves to reflect the new amounts of both assets in the pool.


### Conditions When Reserves and Balance Match
The reserves (`reserves0`) and the actual balance (`amount`) will exactly match under conditions where no transactions have occurred since the last update of reserves or all transactions have been completely finalized and the blockchain state fully updated. This scenario represents a stable state for the liquidity pool, where the recorded and actual liqudities are synchronized.

### Natural Triggering of the bot Condition
Upon any transaction that updates the liquidity pool (such as adding liquidity, removing liquidity, or executing swaps), the Uniswap V2 pair's reserves are updated to reflect the new state of the pool. These transactions include:

- Adding Liquidity: When liquidity is added, both the token and its pair (e.g., BNB in a BSC-based SATX-BNB pool) are deposited into the pool, and the pool's reserves (reserves0 and reserves1) are updated to match these new amounts.
- Removing Liquidity: Similar to adding liquidity, removing liquidity updates the reserves to decrease them in proportion to the amount of liquidity removed.
- Executing Swaps: Swaps between the tokens in the pool adjust the reserves according to the swap amount and the current pool ratio.

### When wont they match
When transactions are in progress but have not yet been finalized or recorded in the blockchain, the actual balances might temporarily differ from the recorded reserves. This can happen during high network congestion or delays in block confirmations. Examples include:

- Swaps: If a swap transaction is executed but not yet finalized, the actual token balances (amount) change immediately when the transaction is broadcasted, but reserves0 only updates once the transaction is confirmed.
- Flash Loan Interactions: Transactions involving flash loans or other complex contract interactions that execute several state changes within a single transaction block might not immediately update the reserves.


## Conclusion
The custom transfer logic within the SATX token's contract introduces complexities that affect liquidity pool interactions and could impact user transactions. The redirection of funds under certain conditions to `_tokenOwner` is a critical concern and means that any users interacting with the contract are likely to lose funds. The conditions that trigger this behavior are based on the comparison of the reserves and actual balances, which can be occur naturally or be manipulated/exploited to drain the liquidity pool. 

## References
- [0xNickLFranklin Tweet](https://twitter.com/0xNickLFranklin/status/1780401891714965654)
