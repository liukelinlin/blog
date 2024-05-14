---
layout: post
title:  "Predict User First Purchase Transaction Using Spark ML"
date:   2016-03-30 12:30:20 -0500
categories: Engineering
---
As an online shopping business, can we predict a user's first purchase, and how?

This article will experiment data pipeline and one of popular ML classification algorithms(Random Forrest) to solve the problem.

# 1. Background

In a shopping site,  user will signup with their dob, gender, city etc, then a series of actions happened after such as merchandize view, add favourite, search etc before first purchase.
* user's demography: age , gender, city
* user's behaviour: searches, views, favourites

After ETL, a user's profile data looks like:
| id   | age | gender | city | searches | views | favourites |
| -------- | ------- | ------- | ------- | ------- | ------- | ------- |
| 12345  | 36    |  1 | 16 | 25 | 198 | 3 |
| 12346  | 29    |  2 | 29 | 102 | 256 | 12 |

# 2. Sampled users based on paid user or not(binary classification). Sampling:
|Total	| Paid	| Unpaid |
| 20k	| 8k	| 12k |

# 3. Data modeling and prediction
![Data Pipeline](https://liukelinlin.github.io/images//spark-pipeline-predict-1st-pruchase.jpg)

```scala
val rf = new RandomForestClassifier()
      .setLabelCol("indexedLabel")
      .setFeaturesCol("indexedFeatures")
      .setNumTrees(10)

    val labelConverter = new IndexToString()
      .setInputCol("prediction")
      .setOutputCol("predictedLabel")
      .setLabels(labelIndexer.labels)

    val pipeline = new Pipeline().setStages(Array(labelIndexer, featureIndexer, rf, labelConverter))
    val model = pipeline.fit(trainingData)
    val predictions = model.transform(testData)

    predictions.select("predictedLabel", "label", "features").show(5)
```

# Result: validated prediction on test dataset, it got ~90% correctness.
