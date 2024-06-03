# elec-demand-arg

A simple linear estimator for the electricity demand of distributors in Argentina


## Variables used
* Monthly temperature($t_{mp}$): This variable was included given the fact that heating and air conditioning systems are one of the most important uses of electricity. Even tough electricity is used throughout the country, the source for the data is the [Dirección General de Estadística y Censos de la Ciudad de
Buenos Aires](https://www.estadisticaciudad.gob.ar/eyc/?p=27702), a goverment entity from the city of Buenos Aires. This is because during the research for this project no summary statistic for temperature for the entire country was found and the Buenos Aires Metro Area is the region that uses more power by a large margin. The value used is an average between the minimum and maximum temperatures.
* Average Seasonal Price ($p_e$): This is an inflation adjusted version of the price as published by [CAMMESA](https://cammesaweb.cammesa.com/informe-anual/). The inflation adjustment is done with [this inflation data](https://www.alphacast.io/datasets/inflation-argentina-long-term-cpi-data-monthly-29891).
* Wage Index ($w$): This variable is used as a proxy for the income of the households. The source of the data is [INDEC](https://www.indec.gob.ar/indec/web/Nivel4-Tema-4-31-61) and it is inflation adjusted using the same data as the previous item.
* EMAE ($a$): This is a monthly economic activity indicator produced by [INDEC](https://www.indec.gob.ar/indec/web/Nivel4-Tema-3-9-48), and it's added to take into account the fact that electricity is a production input.
* Time ($t$): This variable was added to take into consideration other trends in electricity consumption that are not included in the other variables, such as: increasing efficiency in home appliances, population increase, adoption of electro-intensive production processes, etc. This is a discrete variable so that the first month of the sample (January-2011) $t=1$, in the second month $t=2$ and so on.
## Electricity Demand Estimation
### Regression Equation
In this model, the relationship between the electricity demand, measured in Kilowatts, and the variables is as following:
```math
\hat{q}_{t} = \hat{\beta}_0 + \hat{\beta}_1t_{mp}^2 + \hat{\beta_2}t_{mp} + \hat{\beta}_3ln(p_e) + \hat{\beta}_4w + \hat{\beta}_5a + \hat{\beta}_6t + \hat{u}_t
```
Where $\hat{q}_t$ is the electricity demand and $ \hat{u}_t $ is the sample estimation error, both during the period $t$. Replacing the betas for their values the equation is:
$$\hat{q}_{t} = 15274960 + 38758 \space t_{mp}^2 -1426205 \space t_{mp} -332004 \space ln(p_e) + 320936 \space w + 15812 \space a + 22256 \space t + \hat{u}_t$$
### Model Validity
#### Theoretical Validity
From a microeconomics perspective it is possible to model the demand of electricity as a sum of the demand of electricity as a service used by households and the demand of electricity as a production input used by businesses. This means that from this standpoint the parameters of the model should meet certain conditions.
From the consumer side the first condition is that $\partial{p_e}/\partial{\hat{q}_{t}} <0$ (that is, that increases in price reduce demand) and the second condition, assuming that electricity is a normal good (or rather, a normal service), is that $\partial{w}/\partial{\hat{q}_{t}} >0$ (increasing income increases demand).
From the producer side the conditions to be met are that $\partial{p_e}/\partial{\hat{q}_{t}} <0$ and $\partial{a}/\partial{\hat{q}_{t}} > 0$ (more economic activity lead to more electricity demand).
In the end we have 3 conditions and all of them are met, because:
```math
$$\partial{p_e}/\partial{\hat{q}_{t}} = \frac{\hat{\beta}_3}{p_e} = \frac{-332004}{p_e} <0 ; \space p_e>0$$
```
```math
$$\partial{w}/\partial{\hat{q}_{t}} = \hat{\beta}_4 = 320936 >0$$
```
```math
$$\partial{a}/\partial{\hat{q}_{t}} = \hat{\beta}_5 = 15812 > 0$$
```
Thus the model is theoretically valid.
#### Internal Validity
##### Ommited Variables
As mentioned previously, from a theoretical perspective the relevant variables related to electricity consumption are the income of end users, quantities produced by companies, the number of residential and commercial/residential users, the prices that both types of users must pay for electricity and temperature. In addition, given that the regression presented here has a temporal component, it is necessary to incorporate data that allows taking into account changes in the behavior of households and companies over time. While in some cases the variables are found directly measured on a monthly basis, in other cases these variables do not exist (or are not published on a monthly basis), so it was necessary to use proxy variables. In particular, the inflation-adjusted wage index has been used to measure the real income of households, the average between the maximum and minimum temperature of the city of Buenos Aires to measure the temperature throughout the country, time to measure the number of users and changes in electricity consumption patterns and the average seasonal price for the prices of electricty paid. It was not necessary to use proxy variables to measure economic activity (since EMAE has been used for this purpose). Given all of this, it seems like the model shouldn't have issues with ommited variables.
##### Functional Form Misspecification
This aspect was analyzed using Ramsey's RESET test, which consists of elaborating a regression that includes the first k (in this case, k = 3) powers of the independent variables and performing an F test on them to evaluate whether they are statistically significant. In the case of the presented model, the p-value of the test is 0.08999, which rejects the alternative hypothesis that the regression is misspecified. In addition, it is verified that all the variables included in the model are statistically significant at 99.99%.
Measurement Errors in the Variables. In principle, no measurement problems are observed in the variables used. However, there are two elements that could possibly threaten the internal validity of the model.
##### Errors-in-Variable Bias
There are 2 potential sources of measurement errors:
* In the period 2011-2016, the average seasonal price and the wage index are adjusted using the CPI produce by the province of San Luis, but this index is not necessarily representative of the price variation at the national level.
* The temperature data used is only representative of the city of Buenos Aires (and, possibly, the Greater Buenos Aires Metropolitan Area). Although a large percentage of the country's electricity demand is concentrated in this geographical region, the correct procedure would have been to use a temperature index for the whole country, but this information when making this model.
##### Sample Selection Bias
The reason why the dataset covers the 2011-2022 timeframe is that the average seasonal price data was available during that period. Because the sampling is based on the availability of data of a regressor this shouldn't cause any bias.   
Even though there is no reason to state that the selected sample is not representative, the fact that not all of the necessary data are available for previous periods does not allow us to state that the selected sample is representative for them.   
It is also possible that in the future there will be structural changes in electricity demand (for example, an increase of the amount of electric vehicles) that will make the estimator q̂t inconsistent, unbiased or a poor predictor of electricity demand.
##### Simultaneous Causality Bias
* The temperature during a period is not determined by the electricity demand during that same period.
* The average seasonal price is determined directly or indirectly by the government's tariff policy, not by the demand for electricity.
* Wages are determined by factors such as labor productivity or the real exchange rate, but they are not related to electricity demand.
* The demand for electricity as a production input  is determined by economic activity, but there is no such relationship in the opposite direction.
* The passage of time is not determined by electricity demand in Argentina.
##### Inconsistency of OLS Standard Errors
###### Heteroskedasticity
To test for heteroskedasticity the Breusch-Pagan test was performed. The p-value of the test was 0.6206, which with a significance level of 0.05 rejects the null hypothesis of the test and leads to the conclusion that the residuals of the model have constant variance.
###### Serial Correlation
To test for this aspect the Durbin-Watson test was used. In this case the d statistic was 1.923 which rejects the hypothesis that there are serial correlation problems with a significance level of 0.05.
#### External Validity
Given the regulatory differences between the different electricity markets in their respective countries, the existing differences in terms of the adoption of electrical appliances and climate control systems by consumers and the greater or lesser preponderance of industries with high electricity consumption, it is unlikely that the model presented in this article can be used in countries other than Argentina.
### Model Results
This regression has an R² = 0.8572 and. It can be observed that the MAPE is 3.15%, where the largest underestimation is -10.70% and the largest overestimation is 12.04%.
### Economic Properties of the Electricity Demand
Given that the presented model has been shown to meet the necessary conditions to be valid from a theoretical and econometric point of view, it is possible to infer some interesting properties of electricity demand in the stabilized market, including:

1. Optimal Temperature for Minimum Electricity Demand:

It is possible to find the value of t* that minimizes electricity demand, which is approximately 18.4 °C.
This suggests that during periods when the average temperature is around 18.4 °C, electricity demand is at its lowest. This information could be useful for policymakers and electricity providers in developing strategies to manage electricity demand and reduce peak consumption.

2. Impact of Temperature on Electricity Demand:

Given that β₂ = 22256, it is possible to state that, on average, electricity demand increases by 22256 KW per month.
This indicates that a one-degree increase in temperature is associated with an average increase of 22256 KW in electricity demand. This highlights the significant impact of temperature on electricity consumption patterns.

3. Relative Responsiveness to Changes in Wages and Economic Activity:

By analyzing the demand elasticities with respect to wages (η₄₃) and economic activity (η₅₃), it is possible to affirm that electricity demand responds more to changes in wages (recorded by the wage index) than to changes in economic activity (recorded by the EMAE) if |η₄₃| > |η₅₃|. This condition is met throughout the entire study period. This finding is consistent with the fact that in this market, most of the electricity demand comes from residential users.
This suggests that changes in wages have a greater impact on electricity consumption patterns compared to changes in overall economic activity. This could be due to the fact that wage increases lead to higher disposable income, which may lead to increased consumption of electricity-intensive goods and services.

4. Inelasticity of Price and Economic Activity:

It can be observed that both the average seasonal price and economic activity are inelastic throughout the entire study period.
This implies that changes in the average seasonal price or economic activity have a relatively small impact on electricity demand. This could be due to the fact that electricity is an essential commodity, and consumers are willing to pay a higher price for it, even if their income or overall economic activity decreases.

Overall, the presented model provides valuable insights into the properties of electricity demand in the stabilized market. The findings have implications for policymakers, electricity providers, and consumers in terms of understanding and managing electricity consumption patterns.
## Subsidies Estimator