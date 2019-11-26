# Sendy-Logistics-Challenge

This challenge aims to predict the estimated time of delivery of orders, from the point of driver pickup to the point of arrival at final destination.

###### The metric for this challenge is : Root Mean Squared Error

And this is a description of our solution which reached the score of **702.42897989931** RMSE and the **5th place** on the private leaderboard.

**A quick EDA will be available alongside the code for the notebook that achieved such a score**

**Final model** :
For this regression task, our final model was an ensemble of 4 types of blended models with weights. 
We used multiple XGBoost models that were bagged using Sklearn's BaggingRegressor and their results were averaged, A number of bagged LightGBM models using BaggingRegressor, a single CatBoost model , and a sklearn's GradientBoostingRegressor. All these models were making predictions over a 10-fold cross validation scheme. The blending weights were chosen following the best performance on the leaderboard.

### Features : 

**Time features** :

We started off by computing time features and interactions between them such as substractions, additions and multiplications, we chose to keep substractions as they were giving the best performance. We also extracted the pickup hours, binned them to better capture the signal of congestion and non-congestion. The usual time features were also extracted such as weekend or not , week of the month , end/start of month. We also tried interaction between the day of week and pickup hour to better represent congestion related to specific days and hours of the week.

**Rider features** :

The rider data was the most interesting part for us in this challenge, because the rider , potentially, contained information on the many outliers in both datasets. 
We started things off by creating interactions between the rider data, we also used target encoding on the rider Id which gave a slight boost. We also computed for each rider its speed meter/second, by multiplying the distance by 1000 and dividing by the target, using that speed, we set up a logical speed limit that a ride shouldn't surpass which was 18 meter/second, all rides above that speed were given the value of 1 on a new feature called 'error_ride'. We then computed a feature that measured how many errors each rider makes compared to all his rides in the train set which gave us a new feature between 0 and 1 called 'error_rate_rider' that helped with the performance very well. We also removed all the outliers and computed for each rider his average speed to get logical speeds, and imputed the anomalous average speeds with the mean.

**Geospatial features** :

This was the most annoying part of the challenge. Personally this was my first time working with coordinates I didn't even use them when i started the comp. I didn't know coordinates as they are could be fed to a model, by the end of this challenge much more could be done with them. We performed our own clustering on all the points pickup/dest in the data using KMeans, we also tried Principal Component Analysis( PCA ). I also tried to create an interaction between long/lat to pinpoint a single point in the map and then tried a an interaction 'from_to' feature to highlight recurring trajectories, but it severly overfit as it was a highly dimensional categorical feature with not so many datapoints per level. We then moved on to external data, we got the sublocations from the Uber data, and computed the distance from each point in the data to every sublocation in the Uber data which resulted in over 300 features. We then proceeded to remove by hand features that were useless following lightgbm's feature importance. We also gathered popular points in Nairobi from Google Maps that we knew were responsible for a lot of congestion and delays in the rides, and we computed every point's distance to those points.

### Things we tried but did not work out so well : 

**Classification** :
Using the binary feature 'error_ride', we approached the problem at start as a binary classification, we tried to predict the outliers ( small time long distance ) by training an XGBoost on the data while focusing on rider data to achieve that and then we used the classifier predictions as a new binary feature for our regression, We also tried to train the regressor only on 'normal' rides, and manually set to 0 rides that were classified to be outliers but it didn't work well. We reached 0.7 f1 score in validation and it gave a small boost to the performance when we used it as a feature but then we felt the model wasn't robust and stable as we were spending too much time tuning the classifier, and we thought that maybe it would perform slightly good on the public lb, and tank on the private which was a huge risk to take. We decided to drop the classification from our pipeline.

**Data Removal** : 
We decided to remove all rides above 5k seconds as they were 'pulling' the predictions higher by looking at the residuals, it gave a good boost in the performance. But we're not sure it did well on the private leaderboard.


This is almost all of our efforts for the past 2 months. It was a fun challenge with interesting data.
