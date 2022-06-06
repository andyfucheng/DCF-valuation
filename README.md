# DCF Valuation
## Summary
I try to build a discounted cash flow valuation model using Yahoo Finance's open API. The calculation process and logics will be described in the following paragraphes. Please note that any figure calculate from models in this project must not be used as the support for any investment and business decision. Due to the fast-changing characteristic of financial market, any type of investment and company have thier own risks. This simple model does not contain all the potential factors and thus, here cannot guarantee the accuracy of created model.

## What is DCF?
Discounted cash flow (DCF) is one the valution approaches that estimates the present value  of an investment using its expected future cash flows. The analysis approximates the present value of an investment, based on projections how much cash flows it will generate in the future, usually with 5 to 10 years span. The present value of company's cash flow is obtained through deducting the future cash flows using a specific discount rate.

## Yahoo Finance's API
Before starting the analysis, please be sure to read the legal disclaimer of yafinance and the usage about the library. Full version can be found at yfinance documnetation page.  
<br/>To use Yahoo Finance's open API to collect needed information, you have to install the yfinance library to your Python kernel first. Through this library, there adapts a Ticker module, which allows users to access company's profile, historical market data, finanaicals statements, big events like stock split and dividend, and so on.  
<br/>To estimate target's future cash flow, we need the historical data for company's total revenue, net income, total cash from operating activities, capital expenditures, cash and long term debt level. All the data above can be obtained through yfinance. 

## P/E ratio 
### Forward P/E ratio 

### Trailing P/E ratio

## Start model building 


Another important parameter in the DCF model is the discount rate. Here in this project, weighted average cost of capital (WACC) is used as our discount rate to covnert the estimated future cash flow and terminal value to the present value at this point. WACC represents the amount of compensation the market (both bonds and equities buyers) are willing to get paid in returns of putting capital to the firm.

## Example
"AAPL"

## Model limitation and disclaimer
This project aims to show the logic behind DCF model and to actually practice it. Again, the calculation does not guaranteee 
figures accuracy and should not be used as the support for any investment and business decisions.

Since this model use the average percentage of profit margin and free cash flow-to-profit margin from past few years to estimate the future cash flows, companies with higher volatility in revenue change or expenditure spending may not be suitable candidates for DCF model. Also, for companies in the early stage or startups who are still in the cash-burning phase, there might exist negative cash flows which is out of DCF model's capability. 
