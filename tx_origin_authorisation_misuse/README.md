# tx.origin misuse
A Victim contract identifies caller by tx.origin instead of msg.sender 
That may allow some external malicious smart contract to act on behalf of origin user and attack Victim contract

## Attack Scenario

Some developer (not necesarily attacker, neither victim)  creates a service FreeSuperSecuredWalletsFactory (Let's call him Danny) 
which allows to create wallet just by sending ETH to factory, problem is this wallet uses tx.origin for authorization

Some user (future victim) uses this service, send 1 ETH (Let's call him Bob) to FreeSuperSecuredWalletsFactory

Now dishonest Drake spot that there is popular service FreeSuperSecuredWalletsFactory which has a bug
deploys FakeAirdrop contract with his address as _beneficiary, and FreeSuperSecuredWalletsFactory address as _factory
And convince Bob to attend the bounty, and contract indeed do Airdrop of some tokens but at the same time calls following code

```javascript
if(listOfWalletsToSteelFrom._wallets(msg.sender)!=address(0)){
	listOfWalletsToSteelFrom._wallets(msg.sender).sendEth(dishonestGuy);
}
```

which drain FreeSuperSecuredWallet of a msg.sender if particular sender owned one

## How to reproduce example

Consider using remix.ethereum.org

Deploy FreeSuperSecuredWalletsFactory
Send Ether to FreeSuperSecuredWalletsFactory from Victim Account

Deploy FakeAirdrop contract passing as parameters

_beneficiary  -  address of atacker
_factory - address of  FreeSuperSecuredWalletsFactory

Send transaction to FakeAirdrop  from Victim Account

In result funds sended to FreeSuperSecuredWalletsFactory will be transfered to _beneficiary

## Real World Example

Mist Wallet in version 0.4.0
https://blog.ethereum.org/2016/06/24/security-alert-smart-contract-wallets-created-in-frontier-are-vulnerable-to-phishing-attacks/

## Mitigations
Never use tx.origin as source of user identity


## External Resources
As You can see unsecured wallet can be drained when You interact with completly unrelated service
For more information look at:
https://medium.com/coinmonks/solidity-tx-origin-attacks-58211ad95514
https://blog.ethereum.org/2016/06/24/security-alert-smart-contract-wallets-created-in-frontier-are-vulnerable-to-phishing-attacks/