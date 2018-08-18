# Oil Money project

As I mentioned before, this is a project inspired by an article that I read. During low volatility period, the article recommended to trade Norwegian Krone as the oil price was bouncing back. To my curiosity, I intended to find out if this idea is plausible. First thing came to my mind was whether the correlation between Norwegian Krone and Brent Crude was substantial. Norway is one of my fav places in Europe. Unlike Qatar or Saudi Arabia or any other gulf countries, it doesn't heavily rely on oil for its gross income. I doubted if oil price is the only factor that affects the exchange rate of NOK. I looked into international trading statistics of Norway. Basically, most trading partners are inside European Union. Prior to stats, I included UK sterling and Euro in my linear regression model. To my surprise, Norway is actually doing a lot of business with US. Thus, the model regressor consisted of EUR, GBP, USD and Brent Crude. After model identification, I had to choose a currency to evaluate NOK. It should be a stable entity with not much connection to Norways economy. I picked the safe haven currency in East Asia, JPY! Hence, all currencies involved in the model would be evaluated by Japanese Yen. Our regressor variables are EURJPY, GBPJPY, USDJPY and Brent Crude in JPY. Our regressand variable is NOKJPY.

In this demonstration, we denoted time before 2017-04-25 as training horizon. We would use that period to fit a good model. From 2017-04-25 to the end of our dataset, that would be denoted as testing period. We would use our model to forecast and do backtesting in testing horizon. The regression result came out as below. We had a pretty high R square. All T stats and F stats were significant. As the summary suggested, there could be multicollinearity (condition number was large). I wouldn't doubt it as Brent Crude and USD should be negatively correlated. Most commodity future contracts are priced in US dollar. When US dollar appreciates or depreciates, the underlying commodity price is going the opposite way. There may be a cointegration relationship between UK sterling and Euro as well.

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/model%20summary.png)

In this case, we could use elastic net regression to implement a penalty function on this multicollinearity problem. Elastic net is a statistics/machine learning technique that consists of Lasso and Ridge regression (more details can be found from https://en.wikipedia.org/wiki/Elastic_net_regularization ). Assuming elastic net correctly estimated the parameters, let us take elastic net estimated NOK to subtract from OLS estimated NOK. From the figure below, we could tell the distribution of the difference is negatively skewed which implies OLS overestimated NOK.

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/ols%20vs%20elastic%20net.png)

The next step, we took the residuals from both estimations and checked their standard deviations. It turned out that elastic net regression delivered a smaller standard deviation. Now we could determine OLS estimation was biased and elastic net estimation would be the better approach for modelling. Before signal generation, we ought to set up thresholds. For common practise in statistical arbitrage, we take one sigma away from our fitted value as thresholds. When actual NOK price goes above the upper threshold, we take a short position cuz we hold the belief that it would come back to its so-called normal status. For lower threshold scenario, it is vice versa. However, the model is based on historical data. Our estimation is most likely to be overfitting. We need to set up other thresholds for stop orders. When the model doesn't work any more, we would clear our positions and exit the trades gracefully. We use the statistical golden rule, 2 sigmas with 95% confidence interval, to do the trick. If NOK deviates 2 sigmas away from our fitted value, we will reverse our positions and pray for the bounce back. What we didn't think of earlier is that there is a strong trend (could be up/down) when our model breaks. The trend is independent of our training horizon. In another word, no matter which period we denote as training horizon, a strong trend always follow for either direction. The figures below are positions prior to our signal generations and the price movement against our fitted value.

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/oil%20money%20positions.png)
![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/fitted%20vs%20actual.png)

What really interests us is why our model doesn't work any more after a while. A detailed causal analysis is basically writing a thesis. Oh, how I hate writing thesis! I would not get into too much of that rigorous and boring empirical analysis. Instead, I intend to bring uo a short but insightful question here. What we can tell is that after 2017/11/15 things have dramatically changed. What could possibly be the reason? Could it be that Saudi and Iran endorsed an extension of production caps on that particular date? Or Donald Trump got elected as POTUS so he would encourage a depreciated US dollar and lift up restrictions on domestic oil business? If we think of NOK as a stochastic process, we decompose NOK into long term trend and short term random disturbance. Well, apparently short term is dominated by Brent Crude. It partially justifies our model. What really drives long term trend though? 

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/brent%20crude.png)

Ta-da, its Euro. As Norway is in EEA, its economic tie with EU totally dominates the long term trend of NOK. From the normalized figure, we clearly see the trend of both NOK and EUR are somewhat correlated. To get a formal conclusion, we need a cointegration test to test all currencies. Nevertheless, there is no Johansen Test in statsmodel.api package. We would have to use Engle-Granger two step test (more details can be found from https://en.wikipedia.org/wiki/Cointegration#Engle–Granger_two-step_method), which is invented by the mentor of my mentor! A Nobel Prize Winner! Well, we can't get any confirmation of cointegration from the test. The regression model itself is not robust. Sometimes we have to use the old fashion way to make a judgement, which is called instinct. Most big organization researchers recalibrate their forecast with policy or consensus views. This is the time that my guts are telling me EUR is the driver of NOKs long term trend PERIOD! (LOL)

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/trend.png)

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/EG%20failed.png)

Well, we are not economists. I won't write a paper on this  (even though I work as a time series econometrician). Then, let us take a look at our portfolio performance. So far so good, we actually made a few bucks from NOK. Interestingly, after we placed the stop order, I decided to add extra position to see what would happen. In the figure below, we could see the profolio when NOK breached the stop loss threshold. The downwards momentum didnt stop until two months later (maybe another major event in financial market occured). 

![alt text](https://github.com/tattooday/quant-trading/blob/master/Oil%20Money%20project/preview/asset%20value.png)

Wow, did we just do a momentum trading strategy! We started to find a statistical arbitrage between oil price and oil producing country forex. What we got in the end is a momentum trading strategy that use statstical modelling to find the entry point. My explanation for our results would be when the model doesn't work any more, there must be some fundamental situations that have changed investors outlooks in NOK forever. I tested this trading strategy on NOK for different periods. It turned out that our momentum trading is much more riskless and profitable than our statistical arbitrage. It is a robust strategy to enter a trend following. But does it work on any other currencies? Unfortunately, Norway is one of the largest oil producing countries with floating FX regime. The rest is US and Russia. But US dollar is a totally different case. It doesn't rely on WTI price at all. Oil only takes up a very small share in whole economy. What about Russian Ruble? Well, I have attached raw data of Russian Ruble. From my tests, I could only complain Russia had been sanctioned by US so many times. Models on Russian Rumble never work. Fundamental situation of Russia is too volatile. Maybe you have a better idea for Russian Ruble. If you do, plz do not hesitate to let me know.