# Credit Derivative Model

 
The credit derivative model serves the purpose of pricing and calculating sensitivities for the credit derivative products which are Credit Default Swaps (CDSs), First-to-Default swaps (FTDs), FirstNofM basket default swaps (FNMs), all level Collateral Debt Obligations (CDOs, CDO2s, and CDO3s), and forward starting CDOs. 

The model covers:

1.	Default sensitivity;
2.	Exact accruals for the clean price;
3.	Monte Carlo (MC) credit spread sensitivity and MC default sensitivity;
4.	Changes to the normal copula semi-closed form solution;
5.	Oscar-Fritz hazard rate curve calibration.

The model serves the purpose of pricing and calculating sensitivities for all credit derivative products, which are Credit Default Swaps (CDSs), First-to-Default swaps (FTDs), FirstNofM basket default swaps (FNMs),  all level Collateral Debt Obligations (CDOs, CDO2s, and CDO3s), and forward starting CDOs. 

The collateral pool of a CDO trade is defined as a set of N reference names, , in which each reference name is described by a credit spread curve  , a recovery rate  , a default time  , and a notional amount  .  The collateral pool is tranched into M tranches with the tranche defined by an attachment point , a detachment point , and a fixed time horizon   with fee payments at interval .  

The default sensitivity is a kind of stress tests matching situations in which a credit default event has occurred or is perceived to be imminent. Within the current market standard modeling framework, a loss given default (  for the ith obligor) is claimed in the event of default. Compared with the initial version, there are three changes in the methodology:

1.	In the new model, a direct shocking and revaluing is employed via “Curve Substitution Task”. In order to simulate a default event for the ith obligor in the collateral pool, the hazard rate curve is shocked such that an instant default is achieved:
	(1)	 .

Then the value of the trade is calculated directly. The default sensitivity of the jth tranche for the ith obligor is found by computing the difference of values between the perturbed scenario and the unperturbed scenario: 

(2)	 	.

2.	For a vanilla CDO trade, the base correlations rather than the compound correlations are used in the computation of the unperturbed PV and perturbed PV. For an FNM trade, a compound correlation is still used in the computation of PVs.  

3.	The default correlation model is changed from a Poisson model to a normal copula model. Instead of solely relying on the MC simulation, the default sensitivity can be calculated via a semi-closed form normal copula model.

The three changes reflect developments in the structured credit derivative modeling in the past two years. Moreover, the new Oscar-Fritz credit library is much more computationally efficient than the previous SH3 platform. This enables us to calculate the sensitivity via direct shocking and revaluing.

Note that in the computation of the default sensitivity of a vanilla CDO trade, the base correlations are held unchanged. Within the current market standard structured credit derivative modeling framework, the base correlation curves are computed via the market quotes of the CDX and Itraxx indexes. For a bespoke CDO trade in which the collateral pool is not the same as that of the indexes, a mapping methodology is employed to calculate the market implied base correlations. Although different mapping methodologies are used and there is no market standard at the present time, all mapping methodologies are a function of the expected loss of the collateral pool. Perturbing a risk factor of the trade would lead to a change in the expected loss, hence a change in the mapped base correlations. From the viewpoint of modeling, fixing base correlations in the computation of the sensitivity must be considered an approximation. 

However, for the default sensitivity case, a default event will incur a large LGD.  It should not be considered a risk measure with a small perturbation, as defined by a conventional sensitivity. It is more appropriately considered a stress test.  The base correlations will be quite different if we re-map using the changed expected losses in the perturbed scenario. On the other hand, the correlation is defined as the likelihood of joint defaults of the obligors in the collateral pool. Hence, a default event is almost certain to trigger co-movement of other risk factors such as the default probability of other obligors. A simple and subjective mapping methodology is not sufficient to capture the effect of correlation in the case of a default event scenario.

In our opinion, the default sensitivities with and without remapped base correlations are two different risk measures.  They reflect different views on the stress testing scenarios. As shown in the next section, a test is designed to assess to the default sensitivities with and without base correlation re-mapping. At the current stage, the model can not handle the default sensitivities with remapped base correlations. It would be better if both measures are implemented.

The normal copula MC model has been approved as a default correlation model in the Oscar-Fritz credit library [3]. In this phase, the model is extended to include the calculation of sensitivities.  Currently the sensitivities are calculated via perturbing the corresponding risk factors and revaluing. 

It is well known that the sensitivity calculation presents both theoretical and practical challenge to the MC simulation. Due to the presence of the MC noise, the sensitivities calculated via MC simulation are not as accurate as those via closed form solution. The following restrictions and limitations are identified and should be enforced:

•	The MC sensitivity should only be used in the case where no closed form solution is available. Table 1 shows the trade types in which the MC sensitivity is employed. The MC sensitivity will be used for FNM trades and variable maturity synthetic CDO (VMS CDO) trades. The MC sensitivity can also be used as a back up model for vanilla CDO trades, when the semi-closed form solution fails. If a vanilla CDO trade possesses a very complicated LGD structure, the normal copula semi-closed form solution may be too slow and occupy too much memory, and hence, become practically useless. The MC simulation does not have any restriction on homogeneity of the LGDs in the collateral pool.  
  
•	When the MC sensitivity is used for the credit spread sensitivity of the vanilla CDO trade, there is a chance that a negative sensitivity is observed due to the MC noise. Those negative values are in the wrong direction and should be floored to zero.  Note that when using the normal copula MC simulation there is no such problem for FNM trades because a compound correlation is used. In this case, for a given random seed the sequence of the random numbers will be the same for the perturbed scenario. The mapped default times of those random paths will always be modified in one direction. Hence a positive change in expected losses can always be expected if we shock the credit spread curve up by any amount. However, for a CDO trade, base correlations are used and there is no way to fix the MC paths as with FNM trades.
		
•	In order to ensure a comfortable convergence of MC sensitivities, a minimum number of MC paths is required.  1MM paths are set for both FNM trades and CDO trades. In the computation of MTM, 1MM paths are required for FNM trades, while only 500,000 paths are needed for CDO trades. To compute the sensitivities of a CDO trade we require a larger number of MC paths.

It should be noted that all the credit spread sensitivities conducted in our tests are calculated by shocking the related credit spread curve 5bps up, which is the default shocking value in Oscar-Fritz.

Table 1. Models Used for Structured Credit Derivative Products
Product Type	Valuation Model	Sensitivity Computation	Comments
FristNofM (FNM)	Normal copula MC	Normal copula MC	No closed form solution
Variable Maturity Synthetic CDO	Normal copula MC	Normal copula MC	No closed form solution
CDO2&3	Normal copula MC	N/A	The CDO2&3 is mapped to a risk equivalent vanilla CDO trade. 
Vanilla CDO	Normal Copula Semi-closed Form 	Normal Copula Semi-closed Form	
Pay-at-End CDO	Normal Copula Semi-closed Form 	Normal Copula Semi-closed Form	
Forward Start CDO	normal copula WMC 	normal copula WMC	


The computation of a clean price is implemented in the submitted model. Assuming a traded spread s, past coupon date , and valuation date t, and the day count convention ACT/360, the fee accrual is calculated by 

 (3)	 .

This is the standard way of computing fee accruals (see https://finpricing.com/lib/FiBondCoupon.html). 

The normal copula semi-closed form solution in the Oscar-Fritz library has been approved [2]. In order for the model to deal with inhomogeneous loss give defaults of the obligors in a collateral pool, a loss grid tolerance ( ) has been set up. In the generation of the loss grid points, if the difference between an added loss point and the existing loss grid point falls within the amount of   , the added loss point is absorbed into the existing loss grid point and the associated loss probability is directly assigned to the existing loss grid point. 

In the new method, the loss grid is generated in the same way as before. However, the method of assigning loss probability has changed. The computation of the loss probability density for each market factor involves the following procedure:

1.	The standard loss recursion algorithm is employed to build the probability of each loss amount in the grid;
2.	As each name is added a probability is calculated for losses that may or may not be close to existing points in the grid (if the tolerance is very small then the losses will be close to points in the grid, but if TOL is relatively large then the same losses will not be that close to points in the grid);
3.	Suppose the recursion algorithm gives a probability p for a loss amount L that lies between grid points L1 and L2.  We then assign a probability of w p to point L1, and (1-w) p to point L2, where w = (L2 - L) / (L2 - L1).

The expected loss of the trade, calculated using this weighted probability, is viewed as a second order approximation. It is an improvement over the existing approximation, which is viewed as a first order approximation. In the new method, the implications of this approximation can be classified into three aspects:

1.	The expected loss of the entire collateral pool is precise;
2.	If the detachment point of a base tranche lies between L1 and L2 but there is no loss approximation point between L1 and L2 the expected loss of the base tranche remains precise;
3.	If the detachment point of a base tranche happens to be between L1 and L2 and there is a loss L between L1 and L2, a local approximation exists with the error of the expected loss between [0, pL1] depending on the detachment point and L. The maximum error represents a scenario in which detachment point=L1 and L=L1+.   With the default tolerance setting, , we may end up with mispricing a base tranche with the detachment point deviated by <=0.005%.

As indicated in the next section, several tests are designed to assess the implication of the new approximation and the change of tolerance level.

The foremost task of credit derivative pricing is to model the default probability of the underlying reference entity.  The current industry standard for modelling the default probability of an entity is based on the reduced form approach. The unconditional default probability of an obligor, which is described by the hazard rate curve, is calibrated from market information. CDSs which reference this obligor are used.

The current hazard rate curve calibration model has been vetted and re-reviewed. The submitted model is a new implementation in the Oscar-Fritz credit library. There is no change in the calibration instruments or the calibration methodology. The details of the model can be found in [5].

Two other measures of default sensitivity are employed:

Option 1. This measure constructs a new portfolio in which the perturbed obligor is excluded and the principal of the most junior tranche (usually the first tranche) is reduced by the LGD of the perturbed obligor. In this model, there is absolutely no correlation between the perturbed obligor and the rest of the portfolio. The default sensitivity of the jth tranche for the ith obligor is found by: 

(4)	 .


Option 2. In this alternative measure, the tested obligor is perturbed by applying a 30,000 bps credit spread shock. With current risk free rate level the mean survival time of the tested obligor is about 2.5 months. In this model the default correlation is calculated directly from the modified hazard rate, and MC paths are simulated with the updated information. The default sensitivity for the jth tranche is given by

(5)		 ,

where  is the credit spread of the perturbed  ith obligor.

In order to see the effect of fixing base correlation, the default sensitivity with remapped base correlations is also employed as a comparison model. The current mapping method is used to compute the remapped base correlations. This is known as “scaling” with a scaling factor of 0.5.

In assessing this loss grid approximation, the recursion without an approximation is employed as the benchmark model.  The purpose of the loss grid approximation is to increase the computation efficiency when inhomogeneous LGDs of the underlying collateral pool are encountered. Without this approximation, the recursion can still calculate the loss grid and associated loss probability precisely. However, in the case of a very inhomogeneous LGD collateral pool, a very complicated and large loss grid will be encountered, making the computation too slow.


