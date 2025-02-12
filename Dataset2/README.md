# File and Folder introduction of Dataset2

## Contracts

The folder "Contracts" includes 13,413 Ethereum smart contract source codes and their solc versions in Dataset_2.

## Detection_result

The file "results_dataset2.xlsx" describes the detection result of each method in detail.

## Other vulnerability examples

### Detection of IO vulnerabilities

The *Trader* contract (2 transactions, 1.0E+16Wei) is another example, and it was mistakenly discovered that the code ''orfeed.getExchangeRate'' at line 201 exists an IO vulnerability (line 5 of the below code). Thanks to the precise detection logic of our methods, this code was correctly identified as secure. 

```solidity
contract Trader {
    OrFeedInterface orfeed= OrFeedInterface(...);
    ...
    function getUniswapSellPrice() constant returns (uint256){
    	uint256 currentPrice =  orfeed.getExchangeRate("ETH", "SAI", "SELL-UNISWAP-EXCHANGE", 1000000000000000000);
    	return currentPrice;
    }
    ...
}
```

Another misreport example is the *KlerosGovernor* contract (72 transactions) shown in the below code. The execution state ''transaction.executed'' has been checked before the call.value operation (line 674 of the code) to prevent the reentrancy, yet Oyente identified it incorrectly given the flawed detection pattern.

```solidity
contract KlerosGovernor is Arbitrable {
    ...
    function executeTransactionList(...) public {
        ...
        Transaction storage transaction = submission.txs[i];
        uint expendableFunds = getExpendableFunds();
        if (!transaction.executed && transaction.value <= expendableFunds) {
            bool callResult = transaction.target.call.value(transaction.value)(transaction.data);
            if (callResult == true) {
                require(!transaction.executed, "This transaction has already been executed.");
                transaction.executed = true;
            }
        }
        ...
    }
    ...
}
```

### Detection of UIS vulnerabilities

```solidity
contract BasicToken is ERC20Basic {
    mapping(address => uint256) balances;
    uint256 totalSupply_;
    ...
    function totalSupply() public ... {return totalSupply_;}
    function transfer(address _to, uint256 _value) public returns (bool) {
        ...
        balances[msg.sender] = balances[msg.sender].sub(_value);
        ...
    }
    ...
}
```
As shown in the above code, the variable ''totalSupply\_'' (line 111 of the code) in the function ''totalSupply()'' of the *BasicToken* contract (9 transactions, 1.10E+16Wei) is invoked directly without initializing, which violates the development specification. Also, the contract does not provide a function to initialize the mapping variable ''balances'', so that the function ''transfer()'' cannot work normaly, i.e., the value of ''balances[msg.sender]'' is always zero. 
Similarly, as depicted in the below code, the variable ''airdrop\_times'' in the function ''airDrop()'' of the *CoinAtc* contract (1,079 transactions, 5.3E+16Wei) was invoked, and there is no initialization function. Therefore, we should develop contracts in compliance with the development specifications to ensure the correct contract logic. For these vulnerabilities, our methods can provide warnings, which is impossible with methods such as Slither and Manticore, given the lack of detection patterns for these problems. 

```solidity
contract CoinAtc {
    mapping(address => uint256) airdrop_times;
    ...
    function airDrop( address _beneficiary ) public payable returns (bool)
    {
    	...
        if( air_drop_count > 0 )
        {
        	require(airdrop_times[_beneficiary] <= air_drop_count);
        }
    	...
    }
    ...
}
```

```solidity
contract PoliticoinToken is owned, TokenERC20 {
    uint256 public buyPrice = 63800;  
    ...
    function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner public {
        sellPrice = newSellPrice;
        buyPrice = newBuyPrice;
    }
    function buy() payable public {
        uint amount = msg.value / buyPrice;
        _transfer(this, msg.sender, amount);
    }
    ...
}
```

### Detection of TOD vulnerabilities

This vulnerability refers to inconsistent behavior caused by transaction sequence. As shown in line 2 of the above code (corresponding to line 180 of the code), the critical variable ''buyPrice'' of *PoliticoinToken* contract (82 transactions, 7.0E+16Wei) can be read by the function ''buy()'' and updated by the function ''setPrices()''. When the two transactions that invoke these two functions appear in the same block, the miner can alter the order of transactions to cause an inconsistent price. Another classic TOD vulnerability is existed in the *Hasanah* contract (226 transactions, 5.6E+16Wei), i.e., approved attacks. As shown in line 3 of the below code (corresponding to line 226 of the code), the function ''approve'' can authorize amounts to others, and the user can create a consumption transaction that spends the original authorization token and set a higher gas than the changed authorization transaction, so that he can spend both old and new authorized amounts. Thanks to the execution capability of continuous transactions, these two vulnerabilities were detected by SymX and RSymX, yet they were missed by Park and Oyente. 

```solidity
contract Hasanah is ERC20 {
    ...
    function approve(address _spender, uint256 _value) public returns (bool success) {
    // mitigates the ERC20 spend/approval race condition
    if (_value != 0 && allowed[msg.sender][_spender] != 0) { return false; }
        allowed[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }
    ...
}
```