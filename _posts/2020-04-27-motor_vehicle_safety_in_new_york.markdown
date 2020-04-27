---
layout: post
title:      "Motor Vehicle Safety in New York"
date:       2020-04-27 09:28:38 +0000
permalink:  motor_vehicle_safety_in_new_york
---


Motor vehicle accidents are the leading non-natural cause of death for Americans. The sad part to acknowledge is how many of these motor vehicle accidents are due to human irresponsibility behind the wheel. Because of this, I set out to analyze the available data and find trends in the accidents that result in harm. The objective of my project was to discover what factors contribute to people getting injured in a car accident, to determine what can be done to prevent injury in the case of a crash.

## The Data

![](https://thetiaramisu.files.wordpress.com/2020/04/0_0jissvekgk-h49tl.jpg)

Every year, New York State publishes state data on motor vehicle crashes. The New York State [Open Data Initiative](https://data.ny.gov/) began in 2013, with the goal of laying the foundation for a more open, transparent, and innovative state.

Currently, they have crash data available that was collected from the years 2014, 2015, and 2016. From the source, I saw that there are over 2.2 million entries in the individuals dataset and over 1.6 million entries in the vehicles dataset, spanning across the three years. I will just be working with crash data from the year 2016.

For this project, I merged two datasets that were available on New York State's Open Data Initiative:

1. **[Individuals](https://data.ny.gov/Transportation/Motor-Vehicle-Crashes-Individual-Information-Three/ir4y-sesj)**: This dataset includes information for all individuals involved in motor vehicle accidents in New York in the year 2016. It describes descriptive profile information about the individual, seating position, the safety equipment that was used, and whether or not he was injured in the crash.
2. **[Vehicles](https://data.ny.gov/Transportation/Motor-Vehicle-Crashes-Individual-Information-Three/ir4y-sesj)**: This dataset includes information for all vehicles involved in motor vehicle accidents in New York in the year 2016. It provides descriptive information on the vehicle itself, as well as information pertaining to the circumstances at the time of the accident.

I used the Individuals dataset as the base for my merge, and added the vehicle data for each individual. 71% of the individuals were driving on their own, as they didn't share their vehicle ID with any other individual.

## Modeling

My objective was to make a model that predicts the factors that contribute to the occurrence of injuries in motor vehicle crashes. Of the data, 23% of the individuals experienced an injury. I used SMOTE (Synthetic Minority Over-sampling Technique) to address the class imbalance by generating synthetic data based on feature space similarities between existing instances in the minority class.

I used three different machine learning models to explore the results of each and determine which proved to be the strongest.

### Logistic Regression

I started with a vanilla logistic regression model to classify the data into injured vs. not injured. Vanilla just means that no parameters have been adjusted. It provides a baseline model, to which I can compared the tuned models. The training and test accuracy yielded results of 78.43% and 78.6% accuracy. These results weren't bad, however, the recall score for the True values indicated injury was terrible - 12%.

![](https://thetiaramisu.files.wordpress.com/2020/04/unbalanced-logistic-regression.png)

Recall indicates the percentage of true positives that were found, or in this case the percentage of people who were predicted to be injured out of all those who actually were injured. You can see this in the confusion matrix above. Recall takes the number of true positives (those who were predicted to be injured and were injured) divided by the sum of true positives and false negatives (everyone the model indicated was injured, whether they actually were or not).

This was definitely due to the imbalance of the classes, with the low percentage of individuals who were injured. I addressed this by using SMOTE as mentioned above.

After balancing the data, my accuracy dropped to 63.02% for training accuracy and 63.1% for test accuracy. These numbers were worse, but the recall score for the True values was much better, with 65% accuracy (compared to 12%!).

### Random Forests

The random forests algorithm is an ensemble learning method consisting of a large number of decision trees. A single decision tree makes the choice that maximizes the information gain at every step, which creates a problem in that the same results will be given every time, and the performance cannot be improved. Random forests enhances this by taking an aggregate of many decision trees to create high variance, to provide greater depth of understanding to our model.

This method relies on two techniques to achieve high variance among all the trees: bagging and subspace sampling. Bagging, or Bootstrap Aggregation, takes samples of the data with replacement to build each individual tree as we train the data. Subspace sampling is the process of randomly selecting a subset of feature to use as predictors for each node when training a decision tree, rather than using all predictors available at each node. The combination of these two methods allows the random forest to be filled with a diverse set of decision trees, making the model resistant to noise and variance in the data.

```
# Instantiate
forest = RandomForestClassifier()

# train the random forest
forest.fit(X_train, y_train)

# predict
train_preds_forest = forest.predict(X_train)
test_preds_forest = forest.predict(X_test)

# evaluate
train_accuracy_forest = accuracy_score(y_train, train_preds_forest)
test_accuracy_forest = accuracy_score(y_test, test_preds_forest)
report_forest = classification_report(y_test, test_preds_forest)

print("Random Forest")
print("-------------------------")
print(f"Training Accuracy: {(train_accuracy_forest * 100):.4}%")
print(f"Test Accuracy: {(test_accuracy_forest * 100):.4}%")

print("\nClassification report:")
print(report_forest)
print()
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-09-at-2.05.18-am.jpg)

The first random forests model I made yielded a training accuracy of 97.9% and a test accuracy of 80.05%. The extent of this accuracy, especially for the training set in comparison with the testing set, was very worrisome, as it indicated high overfitting to the training data. As I looked at the weight of feature importance, I saw that there was a large overemphasis on the numeric features, and that the model needed tuning. I used a couple different grid searches to find the best estimator parameters given several parameter options I fed each parameter grid. These gave me the best results upon tuning my random forests model.

```
# Create grid    
param_grid = {'n_estimators': [80, 100, 120],
              'criterion': ['gini', 'entropy'],
              'max_features': [5, 7, 9],         
              'max_depth': [5, 8, 10], 
              'min_samples_split': [2, 3, 4]}

# Instantiate the tuned random forest
forest_grid_search1 = GridSearchCV(forest, param_grid, cv=3, n_jobs=-1)

# Train the tuned random forest
forest_grid_search1.fit(X_train, y_train)

# Print best estimator parameters found during the grid search
print(forest_grid_search1.best_params_)
print()
```

*{'criterion': 'gini', 'maxdepth': 10, 'maxfeatures': 9, 'minsamplessplit': 2, 'nestimators': 120}*

Using the results of this parameter grid, I updated my random forests parameter to the following and reran the model. My new results were the following:

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-09-at-2.06.20-am.jpg)

My random forests model yielded a training accuracy of 68.01% and a test accuracy of 67.76% - much lower accuracy than the initial random forests model, but much less dubious. These results were better than those provided by the logistic regression.

### XGBoost

XGBoost is a difference kind of ensemble method. Similar to random forests, boosting algorithms are an ensemble of many different models with high variance. Random forests and boosting algorithms specifically differ in multiple ways, though. Random forests train each tree independently and simultaneously, whereas boosting algorithms train each tree iteratively, identifying weak points and focusing on extrapolating more from those weak points in the next iteration. Boosting algorithms employ a system of weights to determine how important the input for each tree is.

Gradient boosting is an advanced boosting algorithm using gradient descent. This model starts with a weak learner that makes predictions on the dataset. It then calculates the residuals for each data point, combining them to create a loss function, which calculates the overall loss. The algorithm computes the gradient for the loss, given the model's inputs. The gradients and the loss are then used as predictors to train the next tree against. XGBoost is the version of gradient boosting with the highest performance - hence it is rightly named eXtreme Gradient Boosting.

```
# instantiate
booster = xgb.XGBClassifier()

# train the xgboost
booster.fit(X_train, y_train)

# predict
train_preds = booster.predict(X_train)
test_preds = booster.predict(X_test)

# evaluate
train_accuracy_booster = accuracy_score(y_train, train_preds)
test_accuracy_booster = accuracy_score(y_test, test_preds)
report_booster = classification_report(y_test, test_preds)

print("XGBoost")
print("-------------------------")
print(f"Training Accuracy: {(train_accuracy_booster * 100):.4}%")
print(f"Test Accuracy: {(test_accuracy_booster * 100):.4}%")

print("\nClassification report:")
print(report_booster)
print()
```

![](https://thetiaramisu.files.wordpress.com/2020/04/screen-shot-2020-04-09-at-2.15.06-am.jpg)

My initial XGBoost model gave me promising results already. I went through the process of tuning the parameters using multiple grid searches as earlier.

```
pipe_booster = Pipeline([('clf', xgb.XGBClassifier(random_state=11))])

param_booster = {'clf__n_estimators': [100],
                 'clf__max_depth': [3, 5, 7],
                 'clf__min_child_weight': [1, 3, 5],
                 'clf__gamma': [0.0, 0.1]}

booster_grid_search = GridSearchCV(estimator = pipe_booster,
                                  param_grid = param_booster,
                                  cv=3)

booster_grid_search.fit(X_train, y_train)

# print best estimator parameters found during the grid search
print(f'Best accuracy: {round(booster_grid_search.best_score_,4)*100}%')
print(f'Best parameters: {booster_grid_search.best_params_}')
```

*Best parameters: {'clfgamma': 0.0, 'clfmaxdepth': 7, 'clfminchildweight': 1, 'clfnestimators': 100}*

With these updated parameters, I got the following results.

Training accuracy of 82.8% and test accuracy of 82.68%. These results were significantly better than the previous models!

## Interpretations and Conclusion

XGboost was our strongest predictor by far, ending with a predictive accuracy of 83% and a weighted F1 score of 0.82 after parameter tuning.

Interestingly, the logistic regression and random forest models were more balanced in predicting true positives and negatives for injury. However, despite XGboost having a discrepancy of 0.70 recall for predicting injury to 0.95 recall for predicting no injury, these numbers are still higher than each of the other two models.

This means that the XGBoost model that we have will correctly predict the injury of an individual involved in a motor vehicle crash 70% of the time that he actually is injured. 95% of the time, this model will correctly predict that the individual was not injured in the crash when he indeed was not.

One very helpful tool I was able to use with my XGBoost classifier was the ability to view the ranking of feature importance determined by the model.

```
feat_importances_booster2 = pd.Series(booster2.feature_importances_, index=X.columns)
feat_importances_booster2.nlargest(15).sort_values().plot(kind='barh', color='darkgrey', figsize=(6,4)) 
plt.xlabel('Relative Feature Importance with XGBoost - Tuned');
```

![](https://thetiaramisu.files.wordpress.com/2020/04/download.png)

As we look at the ranking of the features, some of the biggest components in predicting an injury include:

* Prior action
* Vehicle type
* Vehicle age
* Airbag deployment

As I looked back at my EDA and did additional exploring, I made the following conclusions from the data.

![](https://thetiaramisu.files.wordpress.com/2020/04/download-3.png)

Motor vehicle accidents caused by backing a vehicle does not often result in injury, especially in comparison with other actions.

![](https://thetiaramisu.files.wordpress.com/2020/04/download-2.png)

The percentage of injuries from motor vehicle crashes is incredibly high. Over 80% of motorcycle crashes result in injury. This makes sense, as motorcyclists involved in crashes have far less protection than drivers and passengers in enclosed vehicles. As such, it is important that all motor vehicle operators are well prepped in motorcycle awareness. It is imperative that motorcyclists wear adequate protective gear, and strong considerations need to be taken into determinations for motorcycle license requirements, to be sure that those who get on the road are fully capable.

![](https://thetiaramisu.files.wordpress.com/2020/04/download-4.png)

The age of a vehicle is important to consider. Newer vehicles tend to offer more protection than older vehicles. Because of this, I would recommend that people maintain their cars well by keeping up with the required maintenance for their specific car. I would also recommend that individuals consider trading in their cars for newer ones as the age of the car starts to exhibit itself in deterioration of its condition or performance.

![](https://thetiaramisu.files.wordpress.com/2020/04/download-1.png)

In the case of an airbag deploying, most individuals involved get injured. This can be due to the fact that airbags typically deploy in more serious accidents. Airbags themselves also cause a lot of impact to the individual, which sometimes results in injury.

## Further Research

It was an interesting analysis of the motor vehicle accident data from New York. To delve deeper in, I want to spend more time looking at the motorcycle crashes and injuries, to see if there is more to understand about them how how to protect motorcyclists. Additionally, I think there would be more to explore regarding the build of the motor vehicles involved, and how they deal with the weather and road surface.

Apart from that, there were two other datasets available also related to motor vehicle accidents from Open NY. Those datasets included more specific details about the crash cases as well as the violations that were committed in each reporting.

Thank you for reading! You can check out my full project [here](https://github.com/thetiaramisu/dsc-3-final-project-online-ds-ft-011419).

![](https://thetiaramisu.files.wordpress.com/2020/04/yolo-so-buckle-up-7x4.jpg)
