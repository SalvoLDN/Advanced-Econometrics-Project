# Advanced-Econometrics-Project

* This is a project that was developed for the Advanced Econometrics module of my BSC in Economics with econometrics 
* It contains a Time Series analysis (Impact of TCU on Inflation Rate) and a Panel Data analysis (Causes of Maternal Mortality across countries)
* Tools: R and RMarkdown
* Data source: 
* TIME SERIES->Database fetched directly from the Fred Website in the beginning of May 2022 (If you run this code now, results might be different from my interpretation as when you fetch data directly from the website you get an up-to-date version) 
* PANEL DATA-> World Bank Data
* Techniques employed: 
* TIME SERIES ANALYSIS-> models run both in levels and first differences, Unit Root tests (ADF,PP,KPSS), cointegration tests (Engle-Granger, Johansen), ARDL model and PSS bounds test, diagnosis tests (Shapiro-Wilk, Breusch-Godfrey, Jarque-Bera) and Impulse Response Functions, VAR and Granger-Causality test, Cholesky-Decomposition and IRFs, ARIMA
* PANEL DATA ANALYSIS-> random effects (RE) model, fixed effects (FE) model using the ‘within’ estimator, pooled OLS and test for poolability, Hausman test, fixed effects model with individual effects only time effects only and with two-way effects, diagnostic tests on the residuals of each model (white noise: auto-correlation and heteroskedasticity), instrumental variable (IV), 2SLS model, GMM model. 
