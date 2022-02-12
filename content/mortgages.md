+++
title = "Mortgages"
date = 2022-02-11
[taxonomies] 
tags=["TIL"]
+++

Mortgages are terrifying.
For most Canadians, a mortgage on their primary home may be the biggest financial transaction in their lives.
It makes sense to try and understand them.

It probably doesn't make sense to code your own mortgage calculator, but it can be [fun][canadian-mortgage].
This is especially true because there are so many [free][mortgage-calculator-public] [calculators][mortgage-calculator-private] online.

Canadian mortgages are tricky because the convention is to advertise as compounded semi-annually, however almost everyone pays them monthly, which means that the rate needs to be converted.

Instead of coding your own [mortgage calculator][canadian-mortgage] ([too][home-buyer] [complicated][land-transfer]) or using someone else's online calculator (not customized enough), you can use [Google Sheets][sheets].

The interest rate must be [converted][compounding-basis] from one compounding basis (twice a year (2)) to another (every month (12)):

```
advertised rate (compounded semi-annually) = AR = 5.0%
advertised rate / 100 = R = 0.05
rate compounded monthly = r = (((1+(R/2))^(2/12))-1)*12 = 0.049486986
```

You can use the [PMT function][pmt] to compute the mortgage, after converting the advertised mortage rate.

```
adjusted rate (from above) = 0.049486986 
number of payments = 25 years * 12 months = 300
principal = 1000000
monthly mortage payment = PMT(adjusted rate/12,number of payments,principal) = $5,816.05
```

[Google Sheets][sheets] allows you to customize formulas while avoiding the error prone aspects of complex financial calculations. 
Most importantly, effortlessly collaborate and share the results with interested parties.

If you don't mind sharing your financial data with a mega corporation, it's very handy.
(Yay! It's 2022 and I have discovered Google Spreadsheets!)
Remember to double-check your results with reputable online calculators.

[canadian-mortgage]: https://github.com/mlbright/canadian-mortgage
[mortgage-calculator-public]: https://itools-ioutils.fcac-acfc.gc.ca/MC-CH/MCCalc-CHCalc-eng.aspx
[mortgage-calculator-private]: https://www.ratehub.ca/mortgage-payment-calculator
[sheets]: https://docs.google.com/spreadsheets
[compounding-basis]: https://en.wikipedia.org/wiki/Compound_interest#Compounding_basis
[pmt]: https://support.google.com/docs/answer/3093185?hl=en
[home-buyer]: https://github.com/mlbright/home-buyer
[land-transfer]: https://github.com/mlbright/toronto-land-transfer-tax