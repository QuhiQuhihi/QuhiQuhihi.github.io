---
title: Sector ETF and Famma-French Factors (Code)
author: Quantitative Boxer
date: 2022-12-15 12:00:00 +0800
categories: [FICC Quant]
categories: [Finance 101]
tags: [finance_101, investment]
mermaid: true
render_with_liquid: false
math: true
---
This post is about sector ETF analysis with Famma French factors.

## Which factors are used in this post?
Factor is statiscal method to analyze impact of variable to data. For example, if many investors follow other investors.   
In this post, I used Famma-French 5 factor + momentum models to decompose sector ETFs. Below is description for factors used in this post. You can download factor data from Kenneth R. French website.
[5_Factors] (https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/Data_Library/f-f_5_factors_2x3.htm)     
[Momentum] (https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/Data_Library/det_mom_factor.html)    

```yaml
SMB (Small Minus Big) : the average return on the nine small stock portfolios minus the average return on the nine big stock portfolios
HML (High Minus Low) : the average return on the two value portfolios minus the average return on the two growth portfolios
RMW (Robust Minus Weak) : the average return on the two robust operating profitability portfolios minus the average return on the two weak operating profitability portfolios,
CMA (Conservative Minus Aggressive) : the average return on the two conservative investment portfolios minus the average return on the two aggressive investment portfolios,
Rm-Rf (Excess return on the market) : value-weight return of all CRSP firms that return data for t minus the one-month Treasury bill rate 
Mom (Momentum) : the average return on the two high prior return portfolios minus the average return on the two low prior return portfolios
```

## Sector ETF used in this post
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


## (code) getting data
you can see full code of this module in here [code] (https://github.com/QuhiQuhihi/Famma-French-Factors-with-Sector-ETF/blob/main/get_rawdata.py)    
```python

class RawDataGetter:

    def __init__(self):

        self.today = datetime.datetime.today()
        self.today_str = self.today.strftime('%Y-%m-%d')

        self.START_DATE = pd.Timestamp('2007-01-03')
        self.END_DATE = pd.Timestamp('2022-12-01')

        self.main_dir = os.getcwd()
        self.data_dir = os.path.join(self.main_dir, 'data',self.today_str)

        self.universe = [
            "IYW", # iShares U.S. Technology ETF|2000-05-15|
            "IYF", # iShares U.S. Financials ETF|2000-05-22|
            "IYZ", # iShares U.S. Telecommunications ETF|2000-05-22|
            "IYH", # iShares U.S. Healthcare ETF|2000-06-12|
            "IYE", # iShares U.S. Energy ETF|2000-06-12|
            "IYK", # iShares U.S. Consumer Staples ETF|2000-06-12|
            "IYG", # iShares U.S. Financial Services ETF|2000-06-12|
            "IYJ", # iShares U.S. Industrials ETF|2000-06-12|
            "IDU", # iShares U.S. Utilities ETF|2000-06-12|
            "IYM", # iShares U.S. Basic Materials ETF|2000-06-12|
            "IYC", # iShares U.S. Consumer Discretionary ETF|2000-06-12|
            "IBB", # iShares Biotechnology ETF|2001-02-05|
            "IGM", # iShares Expanded Tech Sector ETF|2001-03-13|
            "SOXX", # iShares Semiconductor ETF|2001-07-10|
            "IGV", # iShares Expanded Tech-Software Sector ETF|2001-07-10|
            "IGN", # iShares North American Tech-Multimedia Networking ETF|2001-07-10|
            "IGE", # iShares North American Natural Resources ETF|2001-10-22|
            "IYT", # iShares U.S. Transportation ETF|2003-10-06|
            "IHI", # iShares U.S. Medical Devices ETF|2006-05-01|
            "ITA", # iShares U.S. Aerospace & Defense ETF|2006-05-01|
            "IHF", # iShares U.S. Healthcare Providers ETF|2006-05-01|
            "IEO", # iShares U.S. Oil & Gas Exploration & Production ETF|2006-05-01|
            "ITB", # iShares U.S. Home Construction ETF|2006-05-01|
            "IAT", # iShares U.S. Regional Banks ETF|2006-05-01|
            "IAI", # iShares U.S. Broker-Dealers & Securities Exchanges ETF|2006-05-01|
            "IAK", # iShares U.S. Insurance ETF|2006-05-01|
            "IHE", # iShares U.S. Pharmaceuticals ETF|2006-05-01|
            "IEZ"  # iShares U.S. Oil Equipment & Services ETF|2006-05-01|
                ]

    def get_market_date(self):

        # Get Monthly sector ETF data from yahoo finance
        # mkt_data = yf.download(self.universe, start=self.START_DATE,interval="1mo")['Adj Close']

        # Get Daily sector ETF data from yahoo finance
        mkt_data = yf.download(self.universe, start=self.START_DATE, interval="1d")['Adj Close']

        mkt_data.to_csv(os.path.join(self.data_dir,'sector_ETF.csv'))

        return mkt_data
    
    def get_fama_french_data(self):

        url = \
            "http://mba.tuck.dartmouth.edu/pages/faculty/ken.french/ftp/" + \
            "F-F_Research_Data_5_Factors_2x3_CSV.zip"
        file_name = "F-F_Research_Data_5_Factors_2x3.csv"
        file = self.get_file_from_url(url, file_name)
        df_ff5 = pd.read_csv(file, index_col=0, skiprows=3)
        df_ff5 = df_ff5.loc[
            self.START_DATE.strftime('%Y%m'):
        ]

        url = \
            "https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/ftp/" + \
            "F-F_Momentum_Factor_CSV.zip"
        file_name = "F-F_Momentum_Factor.CSV"
        file = self.get_file_from_url(url, file_name)
        df_mom = pd.read_csv(file, index_col=0, skiprows=13).iloc[:-1]
        df_mom = df_mom.loc[
            self.START_DATE.strftime('%Y%m'):
        ]

        # save factor data to csv
        save_factor_data = True
        if save_factor_data == True and os.path.exists(self.data_dir) == False:
            os.mkdir(self.data_dir)
            df_ff5.to_csv(os.path.join(self.data_dir, "F-F_Research_Data_5_Factors_2x3.CSV"))
            df_mom.to_csv(os.path.join(self.data_dir, "F-F_Momentum_Factor.CSV"))
        elif save_factor_data == True and os.path.exists(self.data_dir) == True:
            df_ff5.to_csv(os.path.join(self.data_dir, "F-F_Research_Data_5_Factors_2x3.CSV"))
            df_mom.to_csv(os.path.join(self.data_dir, "F-F_Momentum_Factor.CSV"))
        else:
            pass
        return df_ff5, df_mom

    def get_file_from_url(self, url, file_name):

        r = urlopen(url).read()
        file = ZipFile(BytesIO(r))
        file = file.open(file_name)

        return file
    
    def run(self):
        self.get_fama_french_data()
        self.get_market_date()
```

## Process raw data
you can see full code of this module in here [code] (https://github.com/QuhiQuhihi/Famma-French-Factors-with-Sector-ETF/blob/main/proc_rawdata.py)    
```python
class RawDataProc:
    def __init__(self):

        self.today = datetime.datetime.today()
        self.today_str = self.today.strftime('%Y-%m-%d')
        self.START_DATE = pd.Timestamp('2007-01-01')
        self.END_DATE = pd.Timestamp(self.today)

        self.main_dir = os.getcwd()
        self.data_dir = os.path.join(self.main_dir, 'data',self.today_str)
    
        self.ff5_data = pd.read_csv(
            os.path.join(self.data_dir, 'F-F_Research_Data_5_Factors_2x3.csv') ,index_col=0
        )
        self.mom_data = pd.read_csv(
            os.path.join(self.data_dir, 'F-F_Momentum_Factor.CSV') ,index_col=0
        )
        self.sector_etf_data = pd.read_csv(
            os.path.join(self.data_dir, 'sector_ETF.csv'), index_col='Date', parse_dates=True
        )

    def _process_sector_etf_data(self):
        mkt_data = self.sector_etf_data
        first_date_data = mkt_data.iloc[0:1]
        mkt_data_m = mkt_data.resample('M').last()

        mkt_data = pd.concat([mkt_data_m,first_date_data])
        mkt_data.sort_index(inplace=True)

        mkt_data = mkt_data.ffill()
        mkt_data = 1+np.log(mkt_data/mkt_data.shift(1))
        mkt_data = mkt_data.fillna(1)

        mkt_data.index = mkt_data.index.strftime('%Y%m%d')
        mkt_data = mkt_data.loc[
            self.START_DATE.strftime('%Y%m%d') : self.END_DATE.strftime('%Y%m%d')
        ]
        mkt_data = mkt_data.iloc[1:]

        mkt_data.index = pd.to_datetime(mkt_data.index)
        mkt_data.index = mkt_data.index.strftime('%Y%m')

        mkt_data.to_csv(os.path.join(self.data_dir,'sector_data.csv'))

        return mkt_data

    def _process_factor_data(self):

        ff5_data = self.ff5_data.loc[: ' Annual Factors: January-December ']
        mom_data = self.mom_data.loc[:'Annual Factors:']

        mom_data.rename(columns = {'Mom   ' : 'Mom'}, inplace = True)

        factor = pd.concat([ff5_data.iloc[:-1], mom_data.iloc[:-1]], axis=1)
        factor.to_csv(os.path.join(self.data_dir,'factor_data.csv'))

        return factor
    
    def run(self):
        sector_data = self._process_sector_etf_data()
        factor_data = self._process_factor_data() 

        return sector_data, 
```

## (code) build factor model
you can see full code of this module in here [code] (https://github.com/QuhiQuhihi/Famma-French-Factors-with-Sector-ETF/blob/main/build_factor.py)    
```python
class FactorDecompose:

    def __init__(self):

        self.today = datetime.datetime.today()
        self.today_str = self.today.strftime('%Y-%m-%d')
        self.main_dir = os.getcwd()
        self.data_dir = os.path.join(self.main_dir, 'data',self.today_str)

        self.factors = pd.read_csv(os.path.join(self.data_dir, 'factor_data.csv'), index_col=0)
        self.sectors = pd.read_csv(os.path.join(self.data_dir, 'sector_data.csv'), index_col=0)
        
        self.years = ['2007','2008','2009','2010','2011','2011','2012','2013','2014','2015','2016','2017','2018','2019','2020','2021','2022']
        self.months = ['01','02','03','04','05','06','07','08','09','10','11','12']

        self.factor_list = ['Mkt-RF', 'SMB','HML','RMW','CMA','Mom']

    
    def factor_ols(self, year, ticker, factors_list):
        # select columns for factor data
        rf_data = self.factors[['RF']]
        factor_data = self.factors[['Mkt-RF', 'SMB','HML','RMW','CMA','Mom']]
        sector_data = self.sectors[ticker] - self.factors['RF'] -1

        if year != '2022':
            date_index = []
            for month in self.months:
                date_index.append(int(year+month))
        else:
            date_index = []
            months = ['01','02','03','04','05','06','07','08','09','10']
            for month in months:
                date_index.append(int(year+month))  

        rf_data = rf_data.loc[date_index]
        factor_data = factor_data.loc[date_index]
        sector_data = sector_data.loc[date_index]

        X = factor_data
        y = sector_data

        X = sm.add_constant(X)
        ff_model = sm.OLS(y, X).fit()
        # print(ff_model.summary())

        intercept, b1, b2, b3, b4, b5, b6 = ff_model.params
        
        rf = rf_data['RF'].mean()
        market_premium = factor_data['Mkt-RF'].mean()
        size_premium = factor_data['SMB'].mean()
        value_premium = factor_data['HML'].mean()
        robust_premium = factor_data['RMW'].mean()
        safety_premium = factor_data['CMA'].mean()
        momentum_premium = factor_data['Mom'].mean()

        expected_monthly_return = rf + b1*market_premium + b2*size_premium + b3*value_premium \
                                     + b4*robust_premium + b5*safety_premium + b6*momentum_premium

        print("Ticker is {}".format(ticker))
        print("Expected monthly Return = {}".format(expected_monthly_return))
        print("Expected Annual Return  = {}".format(expected_monthly_return*12))
        return intercept, b1, b2, b3, b4, b5, b6
    
    def run(self):
        tickers = self.sectors.columns
        factors_list = self.factor_list
        df_column = ['id','ticker','year','Mkt-RF', 'SMB','HML','RMW','CMA','Mom']

        factor_results = pd.DataFrame(columns=df_column)
        factor_results.set_index('id', inplace=True)

        i = 0
        for year in self.years:
            for ticker in tickers:
                factor_append = pd.DataFrame(columns=df_column)
                factor_append.set_index('id', inplace=True)

                intercept, b1, b2, b3, b4, b5, b6 = self.factor_ols(year, ticker, factors_list)

                factor_append.loc[i,'ticker'] = ticker
                factor_append.loc[i,'year'] = year
                factor_append.loc[i,'Mkt-RF'] = b1
                factor_append.loc[i,'SMB'] = b2
                factor_append.loc[i,'HML'] = b3
                factor_append.loc[i,'RMW'] = b4
                factor_append.loc[i,'CMA'] = b5
                factor_append.loc[i,'Mom'] = b6
                print(factor_append)
                factor_results = pd.concat([factor_results,factor_append], axis=0)
                i=i+1
        return factor_results
```