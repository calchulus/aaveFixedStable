# aaveFixedStable
A repo for adapting Aave Protocol to add fixed-term Stable Borrow rates

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
