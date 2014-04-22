CDS
========================================================
Heidi Chen 
s.heidi.chen at gmail.com

David Kane
dave.kane at gmail.com

Yang Lu 
yang.lu2014 at gmail.com

CDS Simple Example
--------------------------------------------------------

```{r}
> library(CDS)
> cds1 <- CDS(TDate = "2014-01-14", parSpread = 32, couponRate = 100)
> summary(cds1)
Contract Type:                      SNAC TDate:                       2014-01-14
Currency:                            USD End Date:                    2019-03-20
Spread:                               32 Coupon Rate:                        100
Upfront:                      -348505.14 Spread DV01:                    5162.57
IR DV01:                           90.88 Rec Risk (1 pct):                 79.12

> cds1Rates <- getRates("2014-01-14")
> cds1Rates[[1]]
   expiry matureDate     rate type
1      1M 2014-02-17  0.00159    M
2      2M 2014-03-17 0.002055    M
3      3M 2014-04-17 0.002368    M
4      6M 2014-07-17 0.003355    M
5      1Y 2015-01-19 0.005681    M
6      2Y 2016-01-17  0.00496    S
7      3Y 2017-01-17  0.00881    S
8      4Y 2018-01-17  0.01322    S
9      5Y 2019-01-17 0.017375    S
10     6Y 2020-01-17 0.020875    S
11     7Y 2021-01-17  0.02384    S
12     8Y 2022-01-17 0.026155    S
13     9Y 2023-01-17 0.028045    S
14    10Y 2024-01-17 0.029695    S
15    12Y 2026-01-17 0.032195    S
16    15Y 2029-01-17 0.034545    S
17    20Y 2034-01-17  0.03655    S
18    25Y 2039-01-17  0.03742    S
19    30Y 2044-01-17  0.03788    S
> t(cds1Rates[[2]])
                 text     
badDayConvention "M"      
mmDCC            "ACT/360"
mmCalendars      "none"   
fixedDCC         "30/360" 
floatDCC         "ACT/360"
fixedFreq        "6M"     
floatFreq        "3M"     
swapCalendars    "none"   
> cds1
CDS Contract 
Contract Type:                      SNAC Currency:                           USD
TDate:                        2014-01-14 End Date:                    2019-03-20
Start Date:                   2013-12-20 Backstop Date:               2013-11-15
1st Coupon:                   2014-03-20 Pen Coupon:                  2018-12-20
Day Cnt:                         ACT/360 Freq:                                 Q

Calculation 
Value Date:                   2014-01-17 Price:                           103.41
Principal:                    -341282.92 Spread DV01:                    5162.57
Accrual:                        -7222.22 IR DV01:                          90.88
Upfront:                      -348505.14 Rec Risk (1 pct):                 79.12
Default Prob:                     0.0276 Default Expo:                6341282.92

Credit Curve 
 Term     Rate
   1M 0.001590
   2M 0.002055
   3M 0.002368
   6M 0.003355
   1Y 0.005681
   2Y 0.004960
   3Y 0.008810
   4Y 0.013220
   5Y 0.017375
   6Y 0.020875
   7Y 0.023840
   8Y 0.026155
   9Y 0.028045
  10Y 0.029695
  12Y 0.032195
  15Y 0.034545
  20Y 0.036550
  25Y 0.037420
  30Y 0.037880
```

CDS To-Do's
--------------------------------------------------------
- calcUpfront.c, one option is hard-coded
- Vignette
- Test cases for getRates.R; there might be a bug regarding obtaining rates for the current day - need to check.

Existing R function files
- calcUpfront.R calculates cash settlement amount from conventional spread
- calcSpread.R calculates conventional spread from upfront
- CDS.R constructs a primitive CDS class object.
- calcSpreadDV01.R calculates spread DV 01
- calcIRDV01.R calculates IR DV 01
- getRates.R obtains rates to build an interest rate curve for CDS calculation
- calcRecRisk01 calculates the RecRisk 01
- defaultProb.R approximates the default probability at time t
- defaultExpo.R calculates the default exposure of a CDS contract
- getDates.R get a set of dates relevant for CDS calculation
- price.R calculates the price of a CDS contract

SNAC
- Standard CDS contract specification [http://www.cdsmodel.com/assets/cds-model/docs/Standard%20CDS%20Contract%20Specification.pdf](http://www.cdsmodel.com/assets/cds-model/docs/Standard%20CDS%20Contract%20Specification.pdf)
- Contracts are always traded with a maturity date falling on one of the four roll dates, March/July/Sept/Dec 20. The maturity date is rounded up to the next roll date. 
- If the maturity date falls on a non-business day, then it's moved to the following business day.
- Coupons are paid on a quarterly basis.
- The size of the coupon payment is calculated on ACT/360. 
- Full first coupon is paid on the next available roll date.
- The protection seller pays the protection buyer accrued interest prportional to the time between the last roll date and trade date.
- Backstop date is the date from which protection is provided. Backstop date = current date - 60 calendar days
- For an all-running contract, the spread quoted is the par spread and it is precisely the coupon that is agreed for the premium leg in return for credit event protection.
- PV01 is the PV of a stream of 1 bp payments at each CDS coupon date.
- MTM = (current par spread - original par spread) * current PV01.
- Upfront payment = (par spread - coupon rate) * PV01
- Quoted spread for low spread (IG) names, and points upfront for high spread (HY) names






Notes
- Fixed payment from the protection buyer is the "premium leg"
- Payment of notional less recovery by the protection seller in the event of a default is the "contingent leg"
- Upfront paymant is often quoted in percent of notional.
- In general, the market still quotes CDS on investment grade names in terms of their running spread, while high yield name are quoted on an upfront basis.
- Typically interested in 5-year protection
- DCC: day count convention. ACT/FixedNumber where ACT is the actual
number of days between two events and the fixed denominator is 360 or
365. ACT/365 is often referred to as ACT/365 Fixed or ACT/365F. More details at http://developers.opengamma.com/quantitative-research/Interest-Rate-Instruments-and-Market-Conventions.pdf
- The cash settlement value (or dirty price from bond lexicon) of a CDS
is the discounted value of expected future cash flows, and ignores the
accrued premium. What is normally quoted is the upfront fee or cash
amount (the clean price), which is simply the dirty price with any
accrued interest added. For a newly issued legacy CDS, there is no
accrued interest (recall, interest accrues from T+1), so dirty and
clean price are the same. (OpenGamma)
- "One further point to note, is that the ISDA model is quite general
about the contact spec- ification. It can be used to price CDSs with
any maturity date (it knows nothing about IMM dates), start date and
payment interval. So the contract specifics are inputs to the model -
for standard contracts, this would be a maturity date on the relevant
IMM date, a start date of the IMM date immediately before the trade
date, and quarterly premium payments." (OpenGamma)
- Standard maturity date are unadjusted – always Mar/Jun/Sep/Dec 20th. ISDAhttp://www.cdsmodel.com/assets/cds-model/docs/Standard%20CDS%20Examples.pdf
  - Example: As of Feb09, the 1y standard CDS contract would protect the buyer through Sat 20Mar10. 
- Coupon payment dates are like standard maturity dates, but business day adjusted following (ISDA)


Deal Section (From Bloomberg manual)
- Trade spread: the spread that the deal was initially struck at.
- Business Day: It controls which country's holiday calendar will be used when adjusting cash flow dates for non-settlement days. It will default to the country of the underlying company. For standard deals, a 5D, non holiday calendar is used.
- Business Day Adjustment: It controls how the cashflow dates are adjusted when they fall on non-settlement days.
- Notional: the principal amount on which the payments are based.
- Freq: the frequency at which the Deal Spread payments are made. Normally Q basis in CDS contracts.
- Pay AI: It determines whether accrued interest is paid on a default. If a company defaults between payment dates, there is a certain amount of accrued payment that is owed to the protection seller. "True" means that this accrued will need to be paid by the protection buyer, "False" otherwise.
- Currency: It specifies the currency for the Market Value, Accrued and DV01. It will default to the currency of the underlying company.
- Day Cnt: The day count method to calculate coupon periods/payments as well as accrued interest.
- Date Gen: The method to generate the coupon dates. I: IMM dates; F: forward from the effective date; B: backward from the maturity date; M: monthly IMM dates.
- Backstop date: 60 day look back from which protection is "effective"


Market Section (From Bloomberg manual)
- Curve Date: It determines which date is used for the values of the benchmark curve. It will default to the current date.
- Recovery rate: Expressed in a decimal.
- Term: Maturity date.
- Pts Upf: Upfront payment with each spread.
- Spread: In bps. It is the Par CDS spread for each maturity.
- Prob: It is the default probability of the underlying reference for each maturity. In other words, it is the cumulative probability that a company will default by a given time. 

Calculator Section (From Bloomberg manual)
- Valuation Date: The date used for all accrued interest and present value calculations. It can be thought of as the settle date for the transaction.
- Cash Settled On: The date on which the cash value of the underlying asset is delivered to satisfy the contract, usually T+3.
- Cash Calculated On: The date on which the cash value of the underlying asset is calculated to satisfy the contract, usually T+3. This differs from the above in that settle date uses a 5D calendar generally, but the actual calculations follow holiday schedules.
- Price: Clean dollar price of the contract. Price = (1 - Principal/Notional)*100. Or Price = 1 0 Pts Upf.
- Principal: Market Value of the deal minus any accrued interest.
- Accrued: Interest accrued to the protection seller. 
- MTM: Principal + Accrued
- Cash Amount: MTM value future valued 3 business days to the Cash Settled on Date.
- Spread DV01: Dollar value change in Market Value if the CDS Spread goes up by 1 bp at every point on the Spread Curve.
- IR DV01: Dollar value change in Market Value if the benchmark interest rate goes up by 1 bp every point on the curve.
- Rec Risk (1%): Dollar value change in Market Value if the recovery rate in the spreads section were increased by 1%.
- Default Exposure: (1-Recovery Rate)*Notional - Principal


