---
title: Analyzing Sector ETF with Famma French factors Result
author: quhiquhihi
  name: unknown
  link: https://github.com/QuhiQuhihi
date: 2022-12-10 12:00:00 +0800
categories: [Finance 101]
tags: [finance_101, investment]
render_with_liquid: false
use_math: true
math: true
---
This post is about sector ETF analysis with Famma French factors.

## What is factors for sector ETF?
If we invest in sector ETF such as Technology sector ETF(IYW), we might question what is important for my fund's performance. If you are qualitative analyst, you might take a look at company's strategy or product's competency. But if you are making decisions based on market data, you might wonder what kind of market events are important to my porfolio. In here, I pointed which factors were important for each sector ETF for each years.  

## Which factors are used in this post?
In this post, I used Famma-French 5 factor + momentum models to decompose sector ETFs. Below is description for factors used in this post.   
```yaml
SMB (Small Minus Big) : the average return on the nine small stock portfolios minus the average return on the nine big stock portfolios
HML (High Minus Low) : the average return on the two value portfolios minus the average return on the two growth portfolios
RMW (Robust Minus Weak) : the average return on the two robust operating profitability portfolios minus the average return on the two weak operating profitability portfolios,
CMA (Conservative Minus Aggressive) : the average return on the two conservative investment portfolios minus the average return on the two aggressive investment portfolios,
Rm-Rf (Excess return on the market) : value-weight return of all CRSP firms that return data for t minus the one-month Treasury bill rate 
Mom (Momentum) : the average return on the two high prior return portfolios minus the average return on the two low prior return portfolios
```

### Sector ETF used in this post
```yaml
IYW  : iShares U.S. Technology ETF (2000-05-15)
IYF  : iShares U.S. Financials ETF (2000-05-22)
IYZ  : iShares U.S. Telecommunications ETF (2000-05-22)
IYH  : iShares U.S. Healthcare ETF (2000-06-12)
IYE  : iShares U.S. Energy ETF (2000-06-12)
IYK  : iShares U.S. Consumer Staples ETF (2000-06-12)
IYG  : iShares U.S. Financial Services ETF (2000-06-12)
IYJ  : iShares U.S. Industrials ETF (2000-06-12)
IDU  : iShares U.S. Utilities ETF (2000-06-12)
IYM  : iShares U.S. Basic Materials ETF (2000-06-12)
IYC  : iShares U.S. Consumer Discretionary ETF (2000-06-12)
IBB  : iShares Biotechnology ETF (2001-02-05)
IGM  : iShares Expanded Tech Sector ETF (2001-03-13)
SOXX : iShares Semiconductor ETF (2001-07-10)
IGV  : iShares Expanded Tech-Software Sector ETF (2001-07-10)
IGN  : iShares North American Tech-Multimedia Networking ETF (2001-07-10)
IGE  : iShares North American Natural Resources ETF (2001-10-22)
IYT  : iShares U.S. Transportation ETF (2003-10-06)
IHI  : iShares U.S. Medical Devices ETF (2006-05-01)
ITA  : iShares U.S. Aerospace & Defense ETF (2006-05-01)
IHF  : iShares U.S. Healthcare Providers ETF (2006-05-01)
IEO  : iShares U.S. Oil & Gas Exploration & Production ETF (2006-05-01)
ITB  : iShares U.S. Home Construction ETF (2006-05-01)
IAT  : iShares U.S. Regional Banks ETF (2006-05-01)
IAI  : iShares U.S. Broker-Dealers & Securities Exchanges ETF (2006-05-01)
IAK  : iShares U.S. Insurance ETF (2006-05-01)
IHE  : iShares U.S. Pharmaceuticals ETF (2006-05-01)
IEZ  : iShares U.S. Oil Equipment & Services ETF (2006-05-01)
```
    
#### "IYW",  iShares U.S. Technology ETF 2000-05-15
![MV](/assets/post_image/finance/factor-sector/IYW.png)
#### "IYF",  iShares U.S. Financials ETF 2000-05-22
![MV](/assets/post_image/finance/factor-sector/IYF.png)
#### "IYZ",  iShares U.S. Telecommunications ETF 2000-05-22
![MV](/assets/post_image/finance/factor-sector/IYZ.png)
#### "IYH",  iShares U.S. Healthcare ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYH.png)
#### "IYE",  iShares U.S. Energy ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYE.png)
#### "IYK",  iShares U.S. Consumer Staples ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYK.png)
#### "IYG",  iShares U.S. Financial Services ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYG.png)
#### "IYJ",  iShares U.S. Industrials ETF 2000-06-12 
![MV](/assets/post_image/finance/factor-sector/IYJ.png)
#### "IDU",  iShares U.S. Utilities ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IDU.png)
#### "IYM",  iShares U.S. Basic Materials ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IDU.png)
#### "IYM",  iShares U.S. Basic Materials ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYM.png)
#### "IYC",  iShares U.S. Consumer Discretionary ETF 2000-06-12
![MV](/assets/post_image/finance/factor-sector/IYC.png)
#### "IBB",  iShares Biotechnology ETF 2001-02-05
![MV](/assets/post_image/finance/factor-sector/IBB.png)
#### "IGM",  iShares Expanded Tech Sector ETF 2001-03-13
![MV](/assets/post_image/finance/factor-sector/IGM.png)
#### "SOXX",  iShares Semiconductor ETF 2001-07-10
![MV](/assets/post_image/finance/factor-sector/SOXX.png)
#### "IGV",  iShares Expanded Tech-Software Sector ETF 2001-07-10
![MV](/assets/post_image/finance/factor-sector/IGV.png)
#### "IGN",  iShares North American Tech-Multimedia Networking ETF 2001-07-10
![MV](/assets/post_image/finance/factor-sector/IGN.png)
#### "IGE",  iShares North American Natural Resources ETF 2001-10-22
![MV](/assets/post_image/finance/factor-sector/IGE.png)
#### "IYT",  iShares U.S. Transportation ETF 2003-10-06
![MV](/assets/post_image/finance/factor-sector/IYT.png)
#### "IHI",  iShares U.S. Medical Devices ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IHI.png)
#### "ITA",  iShares U.S. Aerospace & Defense ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/ITA.png)
#### "IHF",  iShares U.S. Healthcare Providers ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IHF.png)
#### "IEO",  iShares U.S. Oil & Gas Exploration & Production ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IEO.png)
#### "ITB",  iShares U.S. Home Construction ETF 2006-05-01
![MV](/assets/post_image/finance/factor-sector/ITB.png)
#### "IAT",  iShares U.S. Regional Banks ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IAT.png)
#### "IAI",  iShares U.S. Broker-Dealers & Securities Exchanges ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IAI.png)
#### "IAK",  iShares U.S. Insurance ETF 2006-05-01
![MV](/assets/post_image/finance/factor-sector/IAK.png)
#### "IHE",  iShares U.S. Pharmaceuticals ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IHE.png)
#### "IEZ"   iShares U.S. Oil Equipment & Services ETF 2006-05-01 
![MV](/assets/post_image/finance/factor-sector/IEZ.png)

