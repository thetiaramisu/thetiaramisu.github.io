---
layout: post
title:      "Houston Housing After Hurricane Harvey"
date:       2020-04-27 09:45:22 +0000
permalink:  houston_housing_after_hurricane_harvey
---

![](https://thetiaramisu.files.wordpress.com/2020/02/la-houston-slider-photo-1-20170831.jpg)

After living in Houston for the past 7 years, I’ve experienced a modest share of hurricanes and flooding. Every part of the country has at least one natural disaster that they tend to face based on their location and geography. Houston’s is flooding.

In August 2017, Hurricane Harvey devastated the Greater Houston Metroplex, and many areas beyond. I consider myself lucky to have incurred no damage to my house. But that’s the thing – the catastrophic impact seemed to happen so randomly throughout the whole metroplex. In every neighborhood, there were houses that were hit and houses that were spared.

In the aftermath of the flood, I organized a flood relief volunteer group to help impacted homeowners in the community. We gutted houses in different areas across Houston. It was sad to see how many homes and memories were lost or destroyed, and no one expected it.

This was the inspiration for my project. With Greater Houston being so close to the gulf and so prone to heavy waters, I want to see if there is a way to forecast the value and sales of homes as flooding occurs overtime.

Due to the data I was able to collect in the given time, the work in my project ended up being much more heavily focused on MLS analysis than on the impact of flooding. But I consider this just the beginning.

## Data Sources

[HAR.com](http://har.com/) (Houston Association of Realtors) is a comprehensive residential property search website that serves as the source of homes for sale and rentals in the state of Texas. It is widely used by realtors and real estate firms, as well as those seeking homes and home assessments throughout the state. The data gathered from this source is comprised of homes listed for sale from the year 2015 onward, as well as their outcomes.

[FEMA](http://fema.gov/) (Federal Emergency Management Agency) acts to prepare for, prevent, respond to, and recover from national disasters. FEMA provides individual assistance to individuals and families who have sustained losses due to such disasters. Need is assessed and funds are respectively distributed. The data gathered from this site includes all applications for individual housing assistance for nationally declared disasters. The dataset will be narrowed to applications specifically related to Hurricane Harvey.

The data gathered from each of these sources includes information regarding homes in the Greater Houston Metroplex. Eight counties comprise the Greater Houston Metroplex: Brazoria, Chambers, Fort Bend, Galveston, Harris, Liberty, Montgomery, and Waller.

![](https://thetiaramisu.files.wordpress.com/2020/02/geographical-extent-of-greater-houston-its-8-counties-and-its-1067-census-tracts-and.jpg)

## Analysis

Looking into the school districts. Katy ISD is currently #1 rated school district in the Houston area (out of 46 school districts), and #14 in Texas (out of 1,024 statewide school districts). Houston ISD, meanwhile, is ranked one of the lowest in Texas and the lowest ranked major school district in the Houston area.

![](https://thetiaramisu.files.wordpress.com/2020/02/katy-vs.-greater-houston.png?w=290&h=129&zoom=2) 
![](https://thetiaramisu.files.wordpress.com/2020/02/hisd-vs.-greater-houston.png?w=291&h=129&zoom=2)

Looking at the sales trends, you can see that the school district makes a big impact on the selling value of homes in the area. Generally, homes in better school districts have a higher and more stable selling value. This can be seen as the case with Katy ISD.

![](https://thetiaramisu.files.wordpress.com/2020/02/num-sold-by-month-listed.png?w=290&h=131&zoom=2)
![](https://thetiaramisu.files.wordpress.com/2020/02/num-sold-by-month-sold.png?w=290&h=131&zoom=2)

Of the homes sold, the majority were listed and closed in May. The least frequent listing month of homes that sold was in December, and the least frequent closing month of homes that sold was in January.

![](https://thetiaramisu.files.wordpress.com/2020/02/avg-sell-price-by-month-sold.png?w=293&h=131&zoom=2)
![](https://thetiaramisu.files.wordpress.com/2020/02/avg-sell-price-per-sqft-by-month-sold.png?w=287&h=131&zoom=2)

Homes sold in June sold for a higher price on average than any other month in terms of actual price as well as price per square foot. Meanwhile homes sold in January sold for a lower price on average than any other month.

## Models

The modeling aspect of my project was broken into two main parts.

**Model 1:** Predicting flood likelihood

Here, I built a model to determine the likelihood for flooding to occur by zip code, in light of the flood damage impacts from Hurricane Harvey. I feature engineered a column to determine general likelihood based on the number of claims that FEMA received in each zip code and whether or not each claim was confirmed to have flood damage. That information is then fed back into a model made to enhance the expectation that a zip code is likely to flood.

I generated a model that could predict a proclivity toward flooding within a zip code with 71.47% accuracy. Using that model, I feature engineered an additional column to add to the final model to predict MLS success. This column indicated whether or not the listed home was in a flood prone area. To make this column, I took in the predictions provided by my XGBoost model and filtered them through a system that eliminated zip codes that had fewer than 100 filed claims and a result of fewer than 50% confirmed cases of flooding per zip code.

**Model 2:** Predicting if a listed home would sell

The final objective was to predict whether or not a listed home will sell. I took the cleaned MLS data and added the column regarding the flood likelihood depending on which zip code the home is located in from the above model.

The highest performing deep learning models ended up being the neural network model after dropping the low performing features with a test accuracy of 66.74%. The lowest performing features were determined by the XGBoost following the class balancing, and had little to no influence over the overall model performance.

The second highest performing deep learning model was the neural network model using L2 regularization, providing a testing accuracy of 66.71%. L2 regularization is also known as weight decay. This enhanced the baseline neural network model by adding the sum of the squares of all weights in the network to the cost function. This provides the network with the opportunity to learn the smaller weights that are more likely to get overlooked.

The initial XGBoost model I made gave a predictive value of 65.55% accuracy. However, I believe that model was overfit due to class imbalance of 2/3 sold as opposed to not sold results. So I used SMOTE (Synthetic Minority Over-sampling Technique) to address the class imbalance by generating synthetic data based on feature space similarities between existing instances in the minority class.

After trying different models and tuning methods, I was able to produce an XGBoost model that delivered 67.56% accuracy in predicting whether or not a home would sell. This ideal model took the XGBoost model after balancing and removed the 10 features with a weight of 0. This is the model I will use as my final model.

### Recommendations

The biggest indicators of whether or not a home sells are:

1. Age of house when listed
2. List price
3. Lot size
4. Number of stories

Separately, from my research alone, these are additional recommendations I have:

1. Inform Houston citizens in flood-potential areas, so won’t be deceived and surprised when the next hurricane comes.
2. Give information that is updated from old maps.

As a homeowner, I have a vested interest in the impact of flooding on housing. Should I worry for my house, even though the flood risk map tells me I don’t need to? How great is the risk, and how does that feed into my decision to get flood insurance?

It is shocking that the last time flood risk in the Houston Metroplex was mapped was in 2001, after Tropical Storm Allison. The process of updating the official flood risk map has just begun and won’t be done until 2023. Will my house then be considered to have a higher risk of flooding? Will my house value change when the redone maps are completed in 2023? Would it be in my best interest to sell my house before then? My goal is to stay ahead of the publicly released data through my own research.

## Future Work

There are so many directions to go with this data! I feel that I am at the very beginning of my delve in. There is so much more I want to do especially with the vast amount of flood data out there beyond FEMA claims.

* Assign a more meaningful description indicating flood risk, such as a scale
* Use county appraisal data to get information on all Greater Houston homes, not just those on HAR
* Find current flood risk by flood plain maps to use as base flood likelihood
* Work with data of flood impact beyond Hurricane Harvey
* Include a broader span of data with sales preceding 2015
* Use county appraisal data to get more details on each home beyond the features provided by HAR
       * More concrete data regarding each home
       * Home appraisal value over time – can assess change over time, taking into account flooding events
       * Information on building material
* Rather than modeling the chance of a home being sold or not, make a model that predicts how long it will take a home to sell given various criteria

To see my full project, click [here](https://github.com/thetiaramisu/Houston-Housing).

![](https://thetiaramisu.files.wordpress.com/2020/02/la-houston-slider-photo-2-20170831.jpg)
