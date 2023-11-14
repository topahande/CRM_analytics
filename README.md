# CRM Analytics 

Customers are in the core of every business. For any business to make profit, the monetary value that the customers bring into the company should be more than the costs the company pays for acquiring their customers and retaining them afterwards. Therefore, optimizing the customer lifecycle is an important task for companies. Here, I will explain some of the analytical methods that are used by companies for this purpose. These methods use key performance indicators (KPIs) which are obtained through customer data. Some example KPIs are:

- Customer Acquisition Rate (E.g. How many new customers the company gained during a specific period?)
- Customer Retention Rate (E.g. How many customers continued to be customers?)
- Customer Churn Rate (E.g. How many customers stopped their customer relationship with the company?)
- Conversion Rate (E.g. How many of the customers actually buy the product after they see the ad?)



## Customer Segmentation with RFM

Recency, Frequency and Monetary are three important metrics to define a customer journey.

Recency: How recent did the customer make a purchase?  
Frequency: How frequent did the customer make purchases?  
Monetary: How much did the customer purchase?  

Using these metrics, a company can segment their customers into groups and take action based on the customer behaviours, for example, by introducing some campaigns.

In the notebook [customer_segmentation_rfm.ipynb](https://github.com/topahande/CRM_analytics/blob/main/customer_segmentation_rfm.ipynb), I perform a simple customer segmentation based on RFM scores, using the online retail transaction data set for years 2010-2011, which is available at https://archive.ics.uci.edu/dataset/502/online+retail+ii. 


## Customer Lifetime Value Estimation

We can define the customer lifetime value (CLV) as the monetary value a customer brings in to the company during his/her lifecycle in relationship with the company. Estimating the CLV of customers is very important for companies in developing marketing strategies. 

In a simple form, CLV can be computed as

$$\text{CLV}=\frac{\text{Customer Value}}{\text{Churn Rate}} * {\text{Profit Margin}},$$

where

$\text{Customer Value} = \text{Average Order Value} * \text{Purchase Frequency},$

$\text{Average Order Value} = \frac{\text{Total Price}}{\text{Total Transaction}},$

$\text{Purchase Frequency} = \frac{\text{Total Transaction}}{\text{Total Number of Customers}},$

$\text{Churn Rate} = 1 - \text{Repeat Rate},$

$\text{Repeat Rate} = \frac{\text{Number of Customers who made purchase more than once}}{\text{Number of all customers}}.$

However, although this exact computation gives an overall idea for the CLV, it does not reflect individual-level differences for example, between transaction or drop-out rates. Therefore, we can develop a statistical model to estimate the expected CLV for each customer:

$$E(\text{CLV}) = E(\text{Number of transactions}) * E(\text{Average profit per transaction}).$$

Here we will use the BG/NBD (Beta Geometric / Negative Binomial Distribution) model to estimate the number of transactions, and the Gamma Gamma model to estimate the average profit for each customer.

With BG/NBD model, the number of transactions is estimated by modeling two processes: *i)* Transaction Process, *ii)* Dropout Process. 

1. In the __transaction process__, the transactions are modeled probabilistically assuming that the number of transactions made by customer $i$ has a Poisson distribution with parameter $\lambda_i$, where $\lambda_i \sim Gamma (\alpha, \beta)$.

2. In the __droupout process__, the probability of the customer $i$ drops out after transaction $t$ is assumed to have a Geometric distribution with parameter $p_i$, where $p_i \sim Beta(a,b)$.

In Gamma-Gamma model, the monetary value per transaction for customer $i$ ($z_i$) is assumed to be distributed randomly around the customer's average transaction value, that is, $z_i \sim Gamma(\rho,v_i)$ and $\bar{z_i}=\frac{\rho}{v_i}$, where the average transaction values across customers is treated as a random variable through the parameter $v_i$ which is assumed to be distributed with $Gamma(\alpha,\beta)$ distribution.


Finally, the expected customer lifetime value can be estimated by multiplying the expected number of transcations, which is estimated by using the BG/NBD model, and the expected monetary value per transaction, which is estimated by using the Gamma-Gamma model. The parameters in each model can be estimated by maximum likelihood estimation (MLE). The MLE estimates of the BG/NBD model parameters will be functions of three variables: the number of transactions (frequency), the duration of customership of the customer at his/her last transaction time (recency), and the duration of the customership of the customer at the point of analysis. The MLE estimates of the Gamma-Gamma model parameters will be functions of two variables: the number of transactions (frequency), and the observed value per transaction (monetary). The python library ``lifetimes`` allows us to estimate CLV of customers by taking these variables as inputs. In the notebook [clv.ipynb](https://github.com/topahande/CRM_analytics/blob/main/clv.ipynb), I use this library to estimate the customer lifetime values for the customers in the online retail transaction data. 

References  
1. Chen,Daqing. (2019). Online Retail II. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D.  
2. Fader, Peter and Hardie, Bruce and Lee, Ka Lok, Counting Your Customers the Easy Way: An Alternative to the Pareto/Nbd Model. Marketing Science, Vol. 24, pp. 275-284, Spring 2005, Available at: https://ssrn.com/abstract=578087
3. Fader, Peter S., Bruce G. S. Hardie, and Ka Lok Lee (2005), “RFM and CLV: Using Iso-value Curves for Customer Base Analysis,” Journal of Marketing Research, 42 (November), 415–430  
4. Fader, Peter S. and Hardie, Bruce G. S. “The Gamma-Gamma Model of Monetary Value, 2013, Available at: https://www.brucehardie.com/notes/025/gamma_gamma.pdf  

