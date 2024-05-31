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
##### Measurement Errors
There are 2 potential sources of measurement errors:
* In the period 2011-2016, the average seasonal price and the wage index are adjusted using the CPI produce by the province of San Luis, but this index is not necessarily representative of the price variation at the national level.
* The temperature data used is only representative of the city of Buenos Aires (and, possibly, the Greater Buenos Aires Metropolitan Area). Although a large percentage of the country's electricity demand is concentrated in this geographical region, the correct procedure would have been to use a temperature index for the whole country, but this information when making this model.
