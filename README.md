# aaveFixedStable
A repo for adapting Aave Protocol to add fixed-term Stable Borrow rates

Current Borrow Rate Parameters can be found here:
https://medium.com/aave/aave-borrowing-rates-upgraded-f6c8b27973a7

I propose the following formula:
With 13.3 second average block times currently on Ethereum, we can create a 6500block Day and a 45500 block week in which a user locks in the borrow rate for this period of time. 

Thus, using the block times as 6500 blocks and 45500 blocks for 1 day and 1 week terms, respectively, and that the volatility of underlying assets grows with the square root of time,  we can create start with a simple weighted square rooot to give two more different slopes for these borrow rates. 
With current parameters getStableRateSlope1 as 600000000000000000000000000 uint256, and getVariableRateSlope2 as 70000000000000000000000000 uint256, I suggest we treat 2 weeks as "infinite time stable", one month might really be an eternity in crypto, but two weeks will be helpful for a safer parameter here in testing.
```
7 - SQRT(6500)/SQRT(91000) = 7 - 0.267 = 6.733 =  Fixed1dayStableSlope1

7 - SQRT(45500)/SQRT(91000) = 7 - 0.707 = 6.293 =  Fixed1weekStableSlope1
```
Thus, we can have interest rates that are strictly lower than the fully flexible interest rate as outlined here, reducing the spread between fixed and variable. This would mean that 1day rates would be much closer to the variable rates (almost a 70% discount), and 1 week ones would have about a 30% discount compared to current stable rates.


A sample Borrow function as below 
```
 async function borrow() {
    const daiAmountinWei = web3.utils.toWei("1000", "ether").toString()
    const daiAddress = "0x6B175474E89094C44Da98b954EedeAC495271d0F" // mainnet DAI
    **const interestRateMode = 3 // fixed-daily-term stable-rate; select 4 for fixed-weekly-term stable-rate**
    const referralCode = "0"

    try {
      // Make the borrow transaction via LendingPool contract
      const lpAddress = await getLendingPoolAddress()
      const lpContract = new web3.eth.Contract(LendingPoolABI, lpAddress)
      await lpContract.methods
        .borrow(daiAddress, daiAmountinWei, interestRateMode, referralCode)
        .send({ from: myAddress })
        .catch((e) => {
          throw Error(`Error borrowing from the LendingPool contract: ${e.message}`)
        })
    } catch (e) {
      alert(e.message)
      console.log(e.message)
    }
  }
```

Repayment remains the same in the flow

The function **swapBorrowRateMode()** would need to be upgraded to take in one parameter, which is the desired rateMode, either 1 (stable), 2(variable), 3 (fixedday) , or 4 (fixedweek).


