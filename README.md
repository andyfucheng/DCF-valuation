# DCF Valuation
## Summary
We are attempting to build a discounted cash flow valuation model using Yahoo Finance's open API. The calculation process and logic behind are described below. Please note that the figures calculated from the model created in this project should not be used as a basis for investment or business decisions. Due to the rapidly changing financial markets, any investment or business carries a certain amount of risk. This simple and basic model does not include all potential factors. Therefore, we do not guarantee the accuracy of the established model.

## What is DCF?
Discounted cash flow (DCF) is one of the valution approaches that estimates the present value  of an investment using its expected future cash flows. The analysis approximates the present value of an investment, based on projections how much cash flows it will generate in the future, usually with span of 5 to 10 years. The present value of a company's cash flows is obtained by discounting future cash flows using a specific discount rate.

## Yahoo Finance's API
Before starting the analysis, please be sure to read the legal disclaimer of yafinance and the usage about the library. Full version can be found at yfinance documnetation page.  
<br/>To use Yahoo Finance's open API to collect needed information, you have to install the yfinance library to your Python kernel first. Through this library, there adapts a Ticker module, which allows users to access company's profile, historical market data, finanaicals statements, big events like stock split and dividend, and so on.  
<br/>To estimate target's future cash flow, we need the historical data for company's total revenue, net income, total cash from operating activities, capital expenditures, cash and long term debt level. All the data above can be obtained through yfinance. 

## P/E ratio 
Before entering the DCF valuation model, I want to introduce a basic but powerful and common valuation indicators named P/E ratio. This method simply divide the stock price with the company's earnings per share given a specific period like the past 1 year. P/E ratio can also be interpreted as the money investors will pay per share for $1 of earnings. Two types of P/E ratios, forward and trailing P/E , are commonly used in practice.
### Forward P/E ratio 
The forward P/E ratio makes use of the company's futre earnings guidance rather historical figures. Using yfinance library in python, we can easily obtain the trailing P/E ratio by running following code.
```
def forward_pe(symbol):
    ticker = yf.Ticker(symbol)
    todays_data = ticker.history(period='1d')
    current_closing_price = todays_data['Close'][0]
    forward_eps = ticker.info.get('forwardEps')
    forward_pe = current_closing_price/forward_eps
    return forward_pe
```
### Trailing P/E ratio
The trailing P/E depends on past perfomance by dividing the current share price by the aggregated EPS earned over the past 12 months. The company's trailing P/E ratio is stored in the info function in yfinance. To get the figure, we can run the code below.
```
def trailing_pe(symbol):
    ticker = yf.Ticker(symbol)
    trailing_pe = ticker.info.get('trailingPE')
    return trailing_pe
```
It is hasty to say that company with lower P/E ratio is a better investing target. However, if there are two companies with similar financial structure and business coverage, it may be the case that the one with lower P/E ratio is an optimal choice.
## Start model building 
The basic logic behind the DCF model is to use company's historical data for profit margin, free cash flow-to-profit margin, and expected growth rate to estimate future cash flows. Here in this project, I use past few years data to calculate the average net profit margin and free cash flow-to-profit margin, and apply that to get expected future profit and free cash flows.
```
ticker = yf.Ticker(symbol)
info = ticker.info
financials = ticker.financials
cashflow = ticker.cashflow
bs = ticker.balance_sheet
ocf = cashflow.loc['Total Cash From Operating Activities']
cap_exp = cashflow.loc['Capital Expenditures']

netprofitmargin = financials.loc['Net Income']/financials.loc['Total Revenue']

fcf2profitmargin = (ocf+cap_exp)/financials.loc['Net Income']

avgproftmargin = netprofitmargin.mean()

avgfcf2profitmargin = fcf2profitmargin.mean()

revenuegrowth = info.get('revenueGrowth')

expectedrev = pd.Series()
expectedrev = [info.get('totalRevenue')*((1+revenuegrowth)**n) for n in range(0,4)]

expectedprofit = [element*avgproftmargin for element in expectedrev]
    
expectedfcf = pd.Series()
expectedfcf = [profit*avgfcf2profitmargin for profit in expectedprofit]
```
Since our future profit margin and free cash flow estimation are mainly based on the historical performance, this model is desigated for mature firm with stable revenue growth and cash flows.
After calculating the expected future cash flows for next five years, we then set a terminal value to our target company which represents the value of a business beyond the forecasted period. We apply the Gordon growth model to get our terminal value. The formula is $GGM = D(1+g)/k_e-g$ which in our case, $D$ will be the expected cash flow at year 5, $g$ is the perpectual growth rate, usually between 2% to 3%. $k_e$ is the cost of capital (WACC).
```
terminalvalue = expectedfcf[-1]*(1+perpetualgrowth)/(wacc-perpetualgrowth)
```
Another important parameter in the DCF model is the discount rate. In this project, the weighted average cost of capital (WACC) is used as the discount rate to convert future cash flows and terminal values to present values.The WACC is the amount of compensation that the market (both bond and equity buyers) are willing to accept in return for putting capital into the firm.
$$WACC = (E/V * R_e) + [D/V * R_d * (1 - T)].$$ where $E$ is market value of firm's equity , $D$ is market value of firm's debt value, $V = E+D$, $R_e$ is the cost of equity, $R_d$ is cost of debt, and $T$ is corporate tax rate. There are many financial websites do the calculation works like finbox.com and GuruFocus.com which people can utilize.
We then use WACC to discount expected future cash flows (included terminal value) to obtain the current value. Last but not the least, unlevered the value by deducting firms' long term debt and adding back the holding cash. The fair value of our target based on DCF model equals to the calculated equity value divided by total outstanding share.
```
todayvalue = pd.Series()
for n in range(0,4):
    todayvalue.loc[n] = expectedprofit[n]/((1+wacc)**(n+1))
todayvalue[4] = terminalvalue/((1+wacc)**4)

cash = bs.loc['Cash'][0]
debt = bs.loc['Long Term Debt'][0]
equity_value = todayvalue.sum()+cash-debt
    
fairvalue = equity_value/info.get('sharesOutstanding')
```
## Example
We type Apple Inc. ticker, "AAPL" into our model and obtain $22.28$ as AAPL's forward P/E ratio and $23.81$ as its trailing P/E. Next, we enter ticker AAPL and searched AAPL's WACC into our cash flow valuation model. The result is demonstrated below.
```
ticker = 'AAPL'
wacc = 0.078
perpetualgrowth =0.03 #set 3% as our company's perpetual growth rate
date_object = datetime.date.today()
try:
    result = fcf_valuation(ticker,wacc,perpetualgrowth)
    dict = {'ticker':ticker, 'fairprice':result[0], 'currentprice':yf.Ticker(ticker).info.get('regularMarketPrice'), 'profitmargin': ((result[0]/yf.Ticker(ticker).info.get('regularMarketPrice'))-1), 'date': date_object}
    if any(watchlist_fcf['ticker'] == ticker):
        watchlist_fcf = watchlist_fcf.drop(watchlist_fcf[watchlist_fcf.ticker == ticker], axis=1)
        watchlist_fcf = watchlist_fcf.append(dict, ignore_index=True).dropna()
        print(ticker+' data is already updated.')
    else:
        watchlist_fcf = watchlist_fcf.append(dict, ignore_index=True)
except:
    pass
print('Please enter ticker: '+ticker)
print('Please enter ticker\'s WACC: '+wacc)
watchlist_fcf
```
> Please enter ticker: AAPL  
> <br/>Please enter ticker's WACC: 0.078  
> <br/>ticker	fairprice	currentprice	profitmargin	date  
><br/>	AAPL	136.458416	146.14	-0.066249	2022-06-07

There is another similar function named *fcf_valuation_adjusted()*, which gives you option to adjust the figure of profit margin and free cash flow-to-profit margin to your reasonable figures.
## Model limitation and disclaimer
The purpose of this project is to demonstrate the logic of the DCF model and put it into practice. The calculations are not intended to guarantee the accuracy of the figures and should not be used as a basis for any investment or business decision.

Since this model estimates future cash flows using averages of profit margins and free cash flow-to-profit ratios over the past several years, companies with large fluctuations in sales and expenses may not be suitable candidates for the DCF model.In addition, for companies in the early stage or startups who are still in the cash-burning phase, there might exist negative cash flows which exceed DCF model's capability. 
