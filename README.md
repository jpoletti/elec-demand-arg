
# elec-demand-arg

A simple linear estimator for the electricity demand of distributors in Argentina.

## Description of the Problem

The retail electricity market in Argentina is structured in three different segments: generation, transmission and distribution, where the last 2 stages are regulated as regional natural monopolies and the first one uses a merit order model, very similar to what is described [here](https://neon.energy/marginal-pricing). The entity that coordinates the market and the transactions between the different market segments and participants is a public-private enterprise called CAMMESA.

The prices that distributors have to pay for the power their users consume is determined by Secretaría de Energía, an entity of the federal government. Because the pricing structure is usually rather complex, CAMMESA calculates the average seasonal price, which is just the amount billed to the distributors divided by amount of power delivered. Another statistic calculated is the average monomic price, which is the average cost of power during a given period (usually a month).

Given Argentina's chronic inflation issues and the fact that the regulated prices are essentially determined by the ruling party, the price of electrical power paid by distributors has always been lower than the cost of generation and transmission, in order to keep consumer spending on utilities low. The difference is compensated by losses in the companies involved in the electrical system (with the worsening of service quality as a result), subsidies from the federal government or a combination of both.

The subsidies have been a consistent problem for fiscal policy, requiring a significant effort from the federal government, reaching up to 1.9% of GDP in 2016.

Thus, the objective of this work is to create a quantitative model that helps to understand and quantify the cost to sustain the retail electricity market. The approach taken is to build a model that estimates electricity demand in this market and multiply that by the difference between cost and prices to get predictions about the needed subsidies.

## Data Used

This section describes the different sources of data used for this project. All of them have a monthly frequency and cover the period from January 2011 to December 2022.

* Monthly temperature ($t_{mp}$): this variable was included given the fact that heating and air conditioning systems are one of the most important uses of electricity. Even tough electricity is used throughout the country, the source for the data is the [Dirección General de Estadística y Censos de la Ciudad de Buenos Aires](https://www.estadisticaciudad.gob.ar/eyc/?p=27702), a government entity from the city of Buenos Aires. This is because during the research for this project no summary statistic for temperature for the entire country was found and the Buenos Aires Metro Area is the most important region when it comes to electrical power consumption. The value used is an average between the minimum and maximum temperatures.

* Average seasonal price ($p_e$): This is an inflation adjusted version of the price as published by [CAMMESA](https://cammesaweb.cammesa.com/informe-anual/). The inflation adjustment is done with [this inflation data](https://www.alphacast.io/datasets/inflation-argentina-long-term-cpi-data-monthly-29891) using January 2011 as the base period.

* Wage index ($w$): this variable is used as a proxy for the income of the households. The source of the data is [INDEC](https://www.indec.gob.ar/indec/web/Nivel4-Tema-4-31-61) and it is inflation adjusted using the same data as the previous item.

* EMAE ($a$): this is a monthly economic activity indicator produced by [INDEC](https://www.indec.gob.ar/indec/web/Nivel4-Tema-3-9-48), and it's added to take into account the fact that electricity is a production input.

* Time ($t$): this variable was added to take into consideration other trends in electricity consumption that are not included in the other variables, such as: increasing efficiency in home appliances, population increase, adoption of electro-intensive production processes, etc. This is a discrete variable so that in the first month of the sample (January-2011) $t=1$, in the second month $t=2$ and so on.

* Average Monomic Price ($p_m$): the source of this variable is the same as in the case of the average seasonal price. It is also adjusted for inflation, using the same data as before.

## Electricity Demand Estimation

### Regression Equation

In this model, the relationship between the electricity demand, measured in Kilowatts, and the independent variables is as following:

$$
\hat{q}_{t} = \hat{\beta}_0 + \hat{\beta}_1 t_{mp}^2 + \hat{\beta}_2 t_{mp} + \hat{\beta}_3 ln(p_e) + \hat{\beta}_4 w + \hat{\beta}_5 a + \hat{\beta}_6 St + \hat{u}_t
$$

Where $\hat{q}_t$ is the electricity demand and $\hat{u}_t$ is the sample estimation error, both during the period $t$. Replacing the weights for their values (fitted using OLS) the equation is:

$$
\hat{q}_{t} = 15274960 + 38758  \space t_{mp}^2 -1426205  \space t_{mp} -332004  \space ln(p_e) + 320936  \space w + 15812  \space a + 22256  \space t + \hat{u}_t
$$

### Model Validity

#### Theoretical Validity

From a microeconomics perspective it is possible to model the demand of electricity as a sum of the demand of electricity as a service used by households and the demand of electricity as a production input used by businesses. This means that from this standpoint the parameters of the model should meet certain conditions.

From the consumer side the first condition is that $\partial{p_e}/\partial{\hat{q}_{t}} <0$ (that is, that increases in price reduce demand) and the second condition, assuming that electricity is a normal good (or rather, a normal service), is that $\partial{w}/\partial{\hat{q}_{t}} >0$ (increasing income increases demand).

From the producer side the conditions to be met are that $\partial{p_e}/\partial{\hat{q}_{t}} <0$ and $\partial{a}/\partial{\hat{q}_{t}} > 0$ (more economic activity leads to more electricity demand).

Thus we have 3 conditions and all of them are met, because:

$$
\partial{p_e}/\partial{\hat{q}_{t}} = \frac{\hat{\beta}_3}{p_e} = \frac{-332004}{p_e} <0 ; \space p_e>0
$$

$$
\partial{w}/\partial{\hat{q}_{t}} = \hat{\beta}_4 = 320936 >0\
$$

$$
\partial{a}/\partial{\hat{q}_{t}} = \hat{\beta}_5 = 15812 > 0
$$

Therefore the model is theoretically valid.

#### Internal Validity

**Omitted Variables** As mentioned previously, from a theoretical perspective the relevant variables related to electricity consumption are the income of end users, quantities produced by companies, the number of residential and commercial/residential users, the prices that both types of users must pay for electricity and temperature. In addition, given that the regression presented here has a temporal component, it is necessary to incorporate data that allows taking into account changes in the behavior of households and companies over time. While in some cases the variables are found directly measured on a monthly basis, in other cases these variables do not exist (or are not published on a monthly basis), so it was necessary to use proxy variables. In particular, the inflation-adjusted wage index has been used to measure the real income of households, the average between the maximum and minimum temperature of the city of Buenos Aires to measure the temperature throughout the country, time to measure the number of users and changes in electricity consumption patterns and the average seasonal price for the prices of electricty paid. It was not necessary to use proxy variables to measure economic activity (since EMAE has been used for this purpose). Given all of this, it seems like the model shouldn't have issues with omitted variables.

**Functional Form Misspecification** This aspect was analyzed using Ramsey's RESET test, which consists of elaborating a regression that includes the first k (in this case, k = 3) powers of the independent variables and performing an F test on them to evaluate whether they are statistically significant. In the case of the presented model, the p-value of the test is 0.08999, which rejects the alternative hypothesis that the regression is misspecified. In addition, it is verified that all the variables included in the model are statistically significant with $\alpha = 0.01$.

**Errors-in-Variable Bias** There are 2 potential sources of measurement errors:

* Because of the issues with the data from INDEC during 2011-2016, the average seasonal price and the wage index are adjusted using the CPI produced by the province of San Luis, but this index is not necessarily representative of the price variation at the national level.

* The temperature data used is only representative of the city of Buenos Aires (and, possibly, the Greater Buenos Aires Metropolitan Area). Although a large percentage of the country's electricity demand is concentrated in this geographical region, the correct procedure would have been to use a temperature index for the whole country, but this information was not available when making this model.  

**Sample Selection Bias** The reason why the dataset covers the 2011-2022 timeframe is that the average seasonal price data was available during that period. Because the sampling is based on the availability of data of a regressor this shouldn't cause any bias.

Even though there is no reason to state that the selected sample is not representative, the fact that not all of the necessary data are available for previous periods does not allow us to state that the selected sample is representative for them.

It is also possible that in the future there will be structural changes in electricity demand (for example, an increase in the amount of electric vehicles) that will make the estimator $\hat{q}_t$ inconsistent, unbiased or a poor predictor of electricity demand.

**Simultaneous Causality Bias**

* The temperature during a period is not determined by the electricity demand during that same period.

* The average seasonal price is determined directly or indirectly by the government's tariff policy, not by the demand for electricity.

* Wages are determined by factors such as labor productivity or the real exchange rate, but they are not related to electricity demand.

* The demand for electricity as a production input is determined by economic activity, but there is no such relationship in the opposite direction.

* The passage of time is not determined by electricity demand in Argentina.

**Inconsistency of OLS Standard Errors**

**Heteroskedasticity** To test for heteroskedasticity the Breusch-Pagan test was performed. The p-value of the test was 0.6206, which with a significance level of 0.05 rejects the null hypothesis of the test and leads to the conclusion that the residuals of the model have constant variance.   

![heterosk-demand](https://raw.githubusercontent.com/jpoletti/elec-demand-arg/main/plots/heterosk-demand.png)

**Serial Correlation** To test for this aspect the Durbin-Watson test was used. In this case the d statistic was 1.923 which rejects the hypothesis that there are serial correlation problems with a significance level of 0.05.

![serial-corr-demand](https://raw.githubusercontent.com/jpoletti/elec-demand-arg/main/plots/serial-corr-demand.png)

#### External Validity

Given the regulatory differences between the different electricity markets in their respective countries, the existing differences in terms of the adoption of electrical appliances and climate control systems by consumers and the greater or lesser preponderance of industries with high electricity consumption, it is unlikely that the model presented in this article can be used in countries other than Argentina.

### Model Results

This regression has an $R^2$ = 0.8572 and the MAPE is 3.15%, where the largest underestimation is -10.70% and the largest overestimation is 12.04%.

![true-pred-demand](https://raw.githubusercontent.com/jpoletti/elec-demand-arg/main/plots/true-pred-demand.png)

![mape-demand](https://raw.githubusercontent.com/jpoletti/elec-demand-arg/main/plots/mape-demand.png)
### Economic Properties of the Electricity Demand

Given that the presented model has been shown to meet the necessary conditions to be valid from a theoretical and econometric point of view, it is possible to infer some interesting properties of electricity demand in the stabilized market, including:

* Optimal Temperature for Minimum Electricity Demand: it is possible to find the value of $t$ that minimizes electricity demand, which is approximately 18.4 °C.

* Growth of Electricity Demand Over Time: given that $\beta_6$ = 22256, it is possible to state that, on average, electricity demand increases by 22256 KW per month.

* Relative Responsiveness to Changes in Wages and Economic Activity: by analyzing the demand elasticity with respect to wages ($\eta_w^q$) and economic activity ($\eta_a^q$), it is possible to affirm that electricity demand responds more to changes in wages (recorded by the wage index) than to changes in economic activity (recorded by the EMAE) if $\eta_w^q >\eta_a^q$ . This condition is met throughout the entire study period. This finding is consistent with the fact that in this market, most of the electricity demand comes from residential users. It also implies that changes in wages have a greater impact on electricity consumption than changes in economic activity.

* Inelasticity of Price and Economic Activity: it can be observed that both the average seasonal price and economic activity are inelastic throughout the entire study period. This implies that changes in the average seasonal price or economic activity have a relatively small impact on electricity demand.

## Subsidies Estimator

The subsidies required by electrical system can be modeled as:

$$s_t = (p_m - p_e)q_t$$

So it's possible to get an estimator of the subsidies by replacing $q_t$ with the demand model:

$$
\hat{s}_t = (p_m - p_e)\hat{q}_t = (p_m - p_e) \left(\hat{\beta}_0 + \hat{\beta}_1t_{mp}^2 + \hat{\beta_2}t_{mp} + \hat{\beta}_3ln(p_e) + \hat{\beta}_4w + \hat{\beta}_5a + \hat{\beta}_6t\right)
$$

### Validity of the Subsidies Estimator

This model is internally valid for the same reasons as the demand estimator. To prove if the model is consistent with the data, a Wald test is performed with the following regression:

$$
\hat{s'}_t = \hat{\gamma}_0 + \hat{\gamma}_1 (p_m-p_e)\hat{q}_t
$$

With the null hypothesis $H_0: \gamma_0 = 0, \gamma_1 = 1$ and the alternative hypothesis $H_a = \gamma_0 \neq 0, \gamma_1 \neq 1$.

If the parameters are fitted using OLS, the regression equation is:

$$
\hat{s'}_t = 32810 + 0.9758 (p_m-p_e)\hat{q}_t
$$

#### Standard Errors of the Subsidies Estimator

To determine whether robust errors should be used it is necessary to evaluate if $\hat{s}_t$ is heteroscedastic and if it has serial correlation issues.

**Heteroskedasticity** In this case it's not necessary to perform any test to confirm that the estimator is heteroscedastic. This can be seen by looking at the definition of the regressor, including the error terms of the demand estimator ($\hat{u}_t$) and of the subsidies estimator ($\hat{v}_t$):

$$
s'_t = \hat{\gamma_0} + \hat{\gamma_1} (p_m-p_e)\left[\hat{\beta}_0 + \hat{\beta}_1t_{mp}^2 + \hat{\beta_2}t_{mp} + \hat{\beta}_3ln(p_e) + \hat{\beta}_4w + \hat{\beta}_5a + \hat{\beta}_6t + \hat{u}_t\right] + \hat{v}_t
$$

Thus, the error in $s'_t$ is equal to $(p_m-p_e)\hat{u}_t + \hat{v}_t$. Given that $\partial s'_t/\partial{(p_m-p_e) > 0}$, this means that the error increases as the predicted subsidies increase, which by definition means that the estimator is heteroscedastic, as shown in the following plot:

![heterosk-subs](https://raw.githubusercontent.com/jpoletti/elec-demand-arg/main/plots/heterosk-subs.png)

**Serial Correlation** Again, to test this we use the Durbin-Watson test. The test statistic ($d$) was 1.797, which again rejects the alternative hypothesis that the regression residuals are correlated with a significance level of 0.05.

![serial-corr-subs](https://github.com/jpoletti/elec-demand-arg/blob/main/plots/serial-corr-subs.png?raw=true)

#### Results of the Validity Test

Given that the results of the previous section, the hypothesis test was performed using HC standard errors. In this case, the p-value was 0.199, which means that the null hypothesis can't be rejected and the subsidies estimator ($s_t = (p_m - p_e)q_t$) is valid.

### Prediction Results for the Subsidies Estimator

Because the subsidies estimator is just the demand estimator multiplied by the difference in prices, the percentage errors are the same as in the demand model.

![true-pred-subs](https://github.com/jpoletti/elec-demand-arg/blob/main/plots/true-pred-subs.png?raw=true)

![mape-subs](https://github.com/jpoletti/elec-demand-arg/blob/main/plots/mape-subs.png?raw=true)

### Economic Properties of the Subsidies Model

* In the case of temperature, wage index, and EMAE, the elasticities of subsidies with respect to these variables ($\eta_{tmp}^s$, $\eta_{w}^s$, and $\eta_{a}^s$, respectively) are equal to the elasticity of electricity demand with respect to the same variables ($\eta_{tmp}^s = \eta_{tmp}^q$, $\eta_{w}^s = \eta_{w}^q$ and $\eta_{a}^s = \eta_{a}^q$), so for these variables it is possible to draw the same conclusions as in the demand estimator.

* The elasticity of subsidies with respect to the average monomial price is $\eta_{p_m}^s = p_m/(p_m - p_e)$. Given that throughout the entire period considered, $p_m > p_e$, the average monomial price is always elastic with respect to energy subsidies. Moreover, it is observed that this variable is the most elastic throughout the studied period.

* The elasticity of subsidies with respect to the average seasonal price is given by the expression $\eta_{p_m}^s = (\hat{q}_t + \hat{\beta}_3p_e) / s_t$. This implies that the elasticity of the average seasonal price with respect to energy subsidies is much greater than the price elasticity of electricity demand, and it even occurs that it is elastic in 41 out of 144 months.