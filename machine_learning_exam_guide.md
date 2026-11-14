# Machine Learning Exam Guide

Study notes for the course lectures in this folder. Each section has a plain-language definition, the comparisons and formulas that show up on exams, a worked example where the lectures use numbers, and a short Spark snippet.

The Spark examples use **Python** and the current DataFrame API (`pyspark.ml`). One lecture slide uses the older RDD API (`pyspark.mllib` with `sc.textFile` and `DecisionTree.trainClassifier`). That older snippet is shown in the tools section so you can recognize it. Everywhere else, learn the DataFrame version: it is the syntax you would actually write.

Assume this exists before the later snippets:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("ml-course").getOrCreate()
```

---

## Contents

1. [What machine learning is](#1-what-machine-learning-is)
2. [Four techniques, supervised and unsupervised](#2-four-techniques-supervised-and-unsupervised)
3. [The machine learning process](#3-the-machine-learning-process)
4. [CRISP-DM](#4-crisp-dm)
5. [KNIME and Spark MLlib](#5-knime-and-spark-mllib)
6. [Data terminology](#6-data-terminology)
7. [Data exploration](#7-data-exploration)
8. [Summary statistics](#8-summary-statistics)
9. [Plots](#9-plots)
10. [Data preparation](#10-data-preparation)
11. [Data quality](#11-data-quality)
12. [Fixing data quality](#12-fixing-data-quality)
13. [Feature selection](#13-feature-selection)
14. [Feature transformation](#14-feature-transformation)
15. [Dimensionality reduction and PCA](#15-dimensionality-reduction-and-pca)
16. [Classification](#16-classification)
17. [Building a model and applying it](#17-building-a-model-and-applying-it)
18. [Generalization, overfitting, underfitting](#18-generalization-overfitting-underfitting)
19. [Decision trees and pruning](#19-decision-trees-and-pruning)
20. [Training, validation, and test sets](#20-training-validation-and-test-sets)
21. [Confusion matrix](#21-confusion-matrix)
22. [Accuracy, precision, recall, F-measure](#22-accuracy-precision-recall-f-measure)
23. [Regression](#23-regression)
24. [Linear regression and least squares](#24-linear-regression-and-least-squares)
25. [Cluster analysis](#25-cluster-analysis)
26. [k-means](#26-k-means)
27. [Association analysis](#27-association-analysis)
28. [Night-before sheet](#28-night-before-sheet)
29. [Questions you should be able to answer](#29-questions-you-should-be-able-to-answer)

---

## 1. What machine learning is

**Machine learning** is learning from data, without writing an explicit rule for every situation. A model finds hidden patterns and supports decisions that come from the data.

It sits between three fields:

- computer science and artificial intelligence (how the algorithms run)
- math and statistics (how patterns and error are measured)
- domain knowledge (what the data means in the real problem)

A **model** learns from data, discovers patterns and trends, and is then used to make a data-driven decision.

Everyday applications from the lectures:

| Application | What the model is doing |
|---|---|
| Credit-card fraud detection | Flag a transaction as fraud or legitimate |
| Handwritten digit recognition | Decide which digit 0–9 a picture shows |
| Recommendations on websites | Suggest items from past behavior |
| Targeted ads | Choose an ad for a user |
| Sentiment analysis | Label text as positive, negative, or neutral |
| Climate monitoring | Find patterns in environmental measurements |
| Crime pattern detection | Find areas or patterns that repeat |
| Drug effectiveness analysis | Relate patient data to how well a drug works |

**Exam point.** A definition that only says “artificial intelligence” is incomplete. The lectures’ definition has four parts: learn from data, no explicit programming, discover hidden patterns, support data-driven decisions.

### Spark

Loading data is the start of “learning from data”:

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
df.printSchema()
df.show(5)
```

---

## 2. Four techniques, supervised and unsupervised

The course uses four techniques.

| Technique | Goal | Target | Supervised? |
|---|---|---|---|
| Classification | Predict a **category** | Categorical label | Yes |
| Regression | Predict a **number** | Numeric value | Yes |
| Cluster analysis | Put similar items into **groups** | None | No |
| Association analysis | Find **rules** linking items that occur together | None | No |

**Supervised learning.** The target (the thing to predict) is provided. The data is **labeled**. Classification and regression are supervised.

**Unsupervised learning.** The target is unknown or unavailable. The data is **unlabeled**. Cluster analysis and association analysis are unsupervised. There is no single “correct answer” column to compare against, so a person has to interpret the result.

### Which technique is this question?

| Question | Technique | Why |
|---|---|---|
| Is the tumor benign or malignant? | Classification | Two categories |
| Will it rain tomorrow? | Classification | A category (yes/no, or sunny/rainy/…) |
| How much rain will fall? | Regression | A number |
| Is this loan low, medium, or high risk? | Classification | Categories, more than two |
| Is this tweet positive, negative, or neutral? | Classification | Categories |
| What test score will this student get? | Regression | A number |
| How much of this product will we sell? | Regression | A number |
| How likely is this drug to work, as a number? | Regression | The lecture lists a numeric likelihood under regression |
| What is tomorrow’s high temperature? | Regression | A number |
| What is a typical house price here? | Regression | A number |
| How much power will be used? | Regression | A number |
| Which customers look alike? | Clustering | Groups, no label given |
| Which weather patterns repeat? | Clustering | Groups |
| Where are the crime hot spots? | Clustering | Groups of locations |
| Which items are bought together? | Association | A rule such as `{bread, milk} => {diapers}` |
| Which web pages are opened in the same session? | Association | Items that co-occur |

**Exam trap.** “Will it rain?” and “how much will it rain?” are different techniques. The first asks for a category. The second asks for a number.

**Exam trap.** Classification and regression are both supervised. The difference is the **type of target**, not whether a target exists.

### Spark

These four imports are the four techniques:

```python
from pyspark.ml.classification import DecisionTreeClassifier   # category
from pyspark.ml.regression import LinearRegression            # number
from pyspark.ml.clustering import KMeans                       # groups
from pyspark.ml.fpm import FPGrowth                            # association rules
```

---

## 3. The machine learning process

The lectures use five steps. The process is a **loop**. A later step often sends you back: the model is weak, the data is dirty, or the original question was incomplete.

```text
ACQUIRE -> PREPARE -> ANALYZE -> REPORT -> ACT
                 ^                  |
                 +------------------+
```

### Acquire

**Goal:** identify and obtain all data related to the problem.

Activities: identify data sources, collect data, integrate data (join sources that do not already live in one table).

### Prepare

Two parts.

**Explore (2-A).** Goal: understand the data. Look for correlations, general trends, and outliers. Describe the data with summary statistics and plots.

**Pre-process (2-B).** Goal: create data that is actually fit for analysis. Three activities: **select**, **clean**, **transform**.

### Analyze

**Goal:** build a model and evaluate the results.

Activities: select a technique (classification, regression, clustering, or association), build the model, assess the results.

### Report

**Goal:** communicate the results and recommend actions. Present the findings to the people who will use them.

### Act

**Goal:** determine actions based on the insights.

Activities: decide the action, implement it, assess the impact. Assessing impact often starts a new loop, because you acquire fresh data about whether the action worked.

**Exam point.** Name the step from its goal. “Understand the nature of the data” is explore. “Create data for analysis” is pre-process. “Build a model and evaluate it” is analyze. “Recommend actions” is report. “Implement and assess impact” is act.

### Spark

Acquire and integrate:

```python
people = spark.read.csv("people.csv", header=True, inferSchema=True)
orders = spark.read.csv("orders.csv", header=True, inferSchema=True)
df = people.join(orders, on="id", how="inner")
```

---

## 4. CRISP-DM

**CRISP-DM** is the **CR**oss **I**ndustry **S**tandard **P**rocess for **D**ata **M**ining. It is a standard, industry-neutral sequence for a data-mining project. Like the five-step process, it is iterative.

| Phase | Goal |
|---|---|
| 1. Business understanding | Define the problem or opportunity, assess the situation, formulate goals |
| 2. Data understanding | Acquire the data and explore it |
| 3. Data preparation | Prepare data for modeling: fix quality, select features, process the values |
| 4. Modeling | Decide what type of problem it is, choose a technique, build the model |
| 5. Evaluation | Assess performance against the success criteria from phase 1 |
| 6. Deployment | Produce the final report, deploy the model, monitor it |

How the two processes line up. There are six CRISP-DM phases and five process steps, so the match is by job, not by a one-to-one count.

| Five-step process | CRISP-DM phase |
|---|---|
| Framing the problem before you collect data | Business understanding |
| Acquire, plus explore | Data understanding |
| Pre-process (clean, select, transform) | Data preparation |
| Analyze: choose a technique and build | Modeling |
| Analyze: assess results | Evaluation |
| Report and act | Deployment |

**Exam point.** Evaluation in CRISP-DM is “does this meet the business success criteria?”, not only “is the accuracy high?”. Deployment includes the report, putting the model into use, and monitoring it afterward.

### Spark

Deployment in code is: save the fitted model, load it later, apply it to new rows.

```python
model.save("models/tree")
# later, in another job:
from pyspark.ml.classification import DecisionTreeClassificationModel
deployed = DecisionTreeClassificationModel.load("models/tree")
deployed.transform(new_data).select("prediction").show()
```

---

## 5. KNIME and Spark MLlib

The course uses two tools for the same ideas.

**KNIME Analytics Platform** is a platform for data analytics, reporting, and visualization.

- GUI, drag-and-drop, interactive
- A **node** does one operation. It has input ports and output ports. A connection links an output port to the next input port.
- A **workflow** is the picture of the whole analysis: nodes wired together.
- The **node repository** lists nodes by category.
- Suited to smaller datasets that fit on one machine and can be explored by clicking.

**Spark MLlib** is a scalable machine learning library that runs on Spark, a distributed computing platform.

- You write code. APIs exist for Scala, Java, Python, and R.
- Algorithms are built to run across a cluster.
- Suited to large datasets.
- The same ideas as KNIME (clean, split, train, evaluate) appear as function calls.

| | KNIME | Spark MLlib |
|---|---|---|
| How you work | Drag and drop nodes | Write code |
| Interface | Graphical workflow | Scala, Java, Python, or R |
| Data size | Smaller datasets | Large datasets |
| Where it runs | Interactive desktop-style platform | Distributed Spark cluster |
| A single step | A node | A function or a pipeline stage |
| The whole analysis | A workflow | A script or a `Pipeline` |

### Spark

The lecture slide shows the older RDD API, roughly:

```python
data = sc.textFile("data.txt")
model = DecisionTree.trainClassifier(parsedData, numClasses=2)
print(model.toDebugString())
model.save(sc, "decisionTreeModel")
```

The same idea in the DataFrame API, which is the one to memorize:

```python
from pyspark.ml.classification import DecisionTreeClassifier
tree = DecisionTreeClassifier(labelCol="label", featuresCol="features", maxDepth=5)
model = tree.fit(train)          # build
preds = model.transform(test)    # apply
print(model.toDebugString)
model.save("decisionTreeModel")
```

---

## 6. Data terminology

A dataset is a table.

- A **sample** is one row: one entity, with a value for every feature. Also called an **instance**, **observation**, **record**, **example**, or **row**.
- A **feature** is one column: one measured property across all samples. Also called a **variable**, **column**, **attribute**, **field**, or **dimension**.
- The **target** (supervised learning only) is the feature the model must predict. The other features are the **inputs**.

| Temperature | Humidity | Wind speed | Weather |
|---|---|---|---|
| 79 | 48 | 2.7 | Sunny |
| 60 | 80 | 3.8 | Rainy |

The first data row is one sample. Temperature, humidity, and wind speed are input features. Weather is the target.

### Types of features

| | Numeric | Categorical |
|---|---|---|
| Also called | Quantitative | Qualitative, nominal |
| Values | Numbers | Labels, names, categories |
| Examples | Height, exam score, transactions per hour, change in stock price | Gender, marital status, customer type, product category, color |
| Typical summaries | Mean, median, standard deviation, correlation | Counts, frequencies, contingency tables |

Other types exist (strings, dates). Most exam questions only need numeric vs categorical.

**Exam point.** “Nominal”, “qualitative”, and “categorical” are the same family in this course. “Quantitative” and “numeric” are the same family.

### Spark

```python
df.printSchema()          # name and type of each feature
print(df.columns)         # feature names
print(df.count())         # number of samples
```

A categorical target has to become a number before most Spark classifiers. `StringIndexer` does that:

```python
from pyspark.ml.feature import StringIndexer
indexer = StringIndexer(inputCol="weather", outputCol="label")
df = indexer.fit(df).transform(df)   # Sunny/Rainy/... -> 0.0, 1.0, ...
```

---

## 7. Data exploration

**Exploratory data analysis (EDA)** means looking at the data before modeling, so you understand it. The goal is data understanding, which leads to an informed analysis.

Two families of techniques:

1. **Summary statistics** — numbers that compress a column.
2. **Visualization** — plots of the same column or of relationships between columns.

What you are looking for:

- **Correlations** — whether two variables move together.
- **Trends** — the general character of the data (rising, falling, repeating).
- **Outliers** — values that sit far away and may signal a problem.

Exploration is where you also check that the table is what you think it is: right number of rows and columns, sensible values, missing entries. Those checks are in the next section.

### Spark

```python
df.describe().show()                       # count, mean, stddev, min, max
df.sample(withReplacement=False, fraction=0.05, seed=1).show(20)
```

`sample` draws a small fraction so you can look at real rows. On a large cluster, explore aggregates and samples. Pulling the full table to your screen defeats Spark.

---

## 8. Summary statistics

A **summary statistic** is a quantity that summarizes and describes a set of values.

| Family | What it describes | Statistics in the lectures |
|---|---|---|
| Location | The central or typical value | Mean, median, mode |
| Spread | How dispersed the values are | Minimum, maximum, range, variance, standard deviation |
| Shape | How lopsided or peaked the distribution is | Skewness, kurtosis |
| Dependence | How two variables move together | Correlation |
| Categorical | How often each label appears | Counts, contingency table |

### Location

- **Mean:** add the values, divide by how many there are. Every value pulls it, including extremes.
- **Median:** the middle value after sorting. If the count is even, average the two middle values. Extremes do not move it much.
- **Mode:** the most frequent value. There can be more than one (bimodal, and so on).

### Spread

- **Range** = maximum − minimum. Easy, and completely determined by the two extremes.
- **Variance** = the average squared deviation from the mean. Large when values sit far from the center. Its units are squared.
- **Standard deviation** = square root of variance. Same units as the data, so it is the one you compare with the values themselves.

### Shape

- **Skewness** measures lopsidedness. Zero is symmetric. **Positive skew** means a longer tail on the high side (the mean is pulled above the median). **Negative skew** means a longer tail on the low side.
- **Kurtosis** measures peakedness / tail weight relative to a bell curve. The age example below is negative, so that sample is flatter than a bell curve.

### Dependence

**Correlation** measures the linear relationship between two numeric variables. It lies between −1 and +1.

- Near +1: the two rise together (strong positive).
- Near −1: one rises as the other falls (strong negative).
- Near 0: little linear relationship. A curve can still exist; correlation only notices straight-line association.

### Categorical data: contingency table

A **contingency table** (cross-tab) counts samples for each combination of two categorical features.

| Color \ Pet | White | Brown | Black | Orange | Total |
|---|---|---|---|---|---|
| Dog | 34 | 44 | 32 | 0 | 110 |
| Cat | 25 | 2 | 43 | 0 | 70 |
| Fish | 1 | 0 | 5 | 33 | 39 |
| Total | 60 | 46 | 80 | 33 | 219 |

Read a cell as a count: 44 dogs are brown. Read a margin as a total: there are 70 cats, 33 orange animals, and 219 animals overall. Column totals and row totals must agree with the grand total. That is a useful arithmetic check.

### Data checks that go with the statistics

**Dimensions.** Number of columns should equal the number of variables you expect. Number of rows should equal the number of samples.

**Values.** Spot-check units and impossible numbers. A temperature column might be Fahrenheit in one row and Celsius in another. A rainfall of 3,678.9 is not a believable daily value. A “date” column might actually be a timestamp.

**Missing values.** How many samples are missing something? Is one feature missing almost everywhere? A feature that is mostly empty is a candidate to drop (see feature selection).

### Worked example: ages

Values: 35, 42, 78, 22, 56, 50, 42, 78, 21, 87. Ten samples.

Sorted: 21, 22, 35, 42, 42, 50, 56, 78, 78, 87.

- Mean = 511 / 10 = **51.1**
- Median = average of the 5th and 6th sorted values = (42 + 50) / 2 = **46**
- Mode = **42 and 78** (each appears twice)
- Range = 87 − 21 = **66**
- Standard deviation ≈ **23.4**, variance ≈ **548.8** (variance is the square of the standard deviation)
- Skewness ≈ **0.30** (mild positive skew: the mean 51.1 sits above the median 46, pulled by 78, 78, and 87)
- Kurtosis ≈ **−1.20**

The lecture’s height/weight example has correlation ≈ **0.89**, a strong positive relationship: taller rows tend to be heavier.

**Exam point.** Given a short sorted list, compute mean, median, and mode by hand. Remember the even-count median and the possibility of two modes.

**Exam point.** Say which family a statistic belongs to. “Correlation” is dependence. “Standard deviation” is spread. “Skewness” is shape. “Median” is location.

### Spark

```python
from pyspark.sql.functions import mean, stddev, skewness, kurtosis, corr, col

df.select(
    mean("age"), stddev("age"), skewness("age"), kurtosis("age")
).show()

df.select(corr("height", "weight").alias("r")).show()
df.crosstab("pet", "color").show()          # contingency table
df.filter(col("rainfall").isNull()).count() # how many missing
```

`describe()` covers count, mean, standard deviation, min, and max. Skewness, kurtosis, correlation, and cross-tabs need the calls above.

---

## 9. Plots

Plots are the visual half of exploration. Use them **with** summary statistics, not as a replacement. The same pictures are reused later to communicate results in the report step.

| Plot | Data it shows | What you read off it |
|---|---|---|
| Histogram | Distribution of **one numeric** variable | Center, skew, outliers. The bars are **bins** (ranges), and the height is how many values fall in that range |
| Line plot | A variable against an index or time | Trend (up/down), cyclical pattern, comparison of several series |
| Scatter plot | **Two numeric** variables, one point per sample | Positive, negative, nonlinear, or no relationship |
| Bar plot | Distribution of a **categorical** variable | How frequent each category is. Grouped bars compare groups side by side. Stacked bars show the parts of a whole |
| Box plot | One numeric variable, easy to compare several | Center, spread, skew, outliers |

### Scatter plot

Each sample is a point. Horizontal position is one feature, vertical position is the other.

- Points rising to the right: positive correlation.
- Points falling to the right: negative correlation.
- A clear curve: a relationship that correlation (a linear measure) will understate.
- A cloud with no slope: little relationship.

### Box plot

The lecture highlights one fact: the **box is the middle 50% of the data**.

- The line inside the box is the **median**.
- The bottom and top of the box are the first and third quartiles. The distance between them is the interquartile range, the middle half of the samples.
- Whiskers extend toward the low and high values.
- Points past the whiskers are drawn as **outliers**.
- **Symmetric:** the median sits in the middle of the box and the whiskers are similar.
- **Positive skew:** longer whisker on the high side.
- **Negative skew:** longer whisker on the low side.

A box plot is the picture you use when the question is “compare these distributions”, not “show every point”.

**Exam point.** Match the chart to the question. “Has this sensor drifted over the month?” is a line plot. “Do height and weight move together?” is a scatter plot. “Is age skewed, and are there outliers?” is a histogram or a box plot. “How many customers are in each category?” is a bar plot.

### Spark

Spark computes the numbers; KNIME is the course’s drag-and-drop way to draw them. A histogram in Spark is the bin edges plus the counts:

```python
values = df.select("age").rdd.map(lambda row: row[0])
edges, counts = values.histogram(8)         # 8 bins; edges has one extra value
print(list(zip(edges[:-1], edges[1:], counts)))

df.approxQuantile("age", [0.25, 0.5, 0.75], 0.0)   # Q1, median, Q3: the box
```

A scatter plot only needs the two columns. On a large table, plot a sample:

```python
df.select("height", "weight").sample(False, 0.02, seed=1).show()
```

---

## 10. Data preparation

**Goal:** create data that is fit for analysis.

Two big jobs:

1. **Cleaning** (also called **data cleansing**) — fix quality problems.
2. **Getting the data into shape** (also called **data wrangling**, **data preprocessing**, or **data munging**) — feature selection and feature transformation, including scaling and dimensionality reduction.

Why it matters: **garbage in, garbage out.** A careful model trained on broken data produces broken conclusions. Preparation is what makes the analysis meaningful.

| Word the lectures use | Same job |
|---|---|
| Data cleaning, data cleansing | Fix missing values, duplicates, inconsistency, noise, outliers |
| Data wrangling, preprocessing, munging | Select and transform features so the table is model-ready |

Activities to be able to list: handle missing values, merge or delete duplicates, repair invalid values, select features (add, remove, combine), transform features (scale, filter, reduce dimensions).

### Spark

A Spark `Pipeline` is the ordered list of preparation and modeling steps. It is the code version of a KNIME workflow.

```python
from pyspark.ml import Pipeline
prep = Pipeline(stages=[imputer, assembler, scaler, tree])
model = prep.fit(train)
```

Each stage is defined in the sections below. The order is: fix missing values, build the feature vector, scale, then fit the model.

---

## 11. Data quality

Real data is messy. The lectures’ quality problems:

| Problem | What it looks like | Example from the lectures |
|---|---|---|
| Missing data | An empty cell | Age or income left blank |
| Duplicate data | The same entity more than once | “Sidney” and “Sid” at the same street address; Kiril at two addresses |
| Invalid data | A value that cannot be right | Zip code `8033A` |
| Noise | Corruption on top of the real value | Garbage characters inside an address or a name |
| Outliers | An extreme value | A sensor failure that records a spike |

Poor data quality produces poor analysis results. That is why these get fixed **before** modeling, during preparation.

Outliers have two roles, and exams mix them up:

- During cleaning, an outlier is often a **fault** (the sensor failed). If that row is not the point of the study, it distorts means, variances, and distances.
- In some studies the outlier **is** the point (fraud, a failing sensor you must find). Then you keep it and study it.

### Spark

Find the problems before you fix them:

```python
from pyspark.sql.functions import col

df.filter(col("income").isNull() | col("age").isNull()).count()
df.groupBy("name", "address").count().filter(col("count") > 1).show()
df.filter(~col("zip").rlike("^[0-9]{5}$")).show()     # zip is not 5 digits
```

---

## 12. Fixing data quality

**Domain knowledge is required.** A reasonable repair depends on what the field means. The same number can be a sensor glitch in one project and the event you care about in another.

### Missing values

**Imputation** means replacing a missing value with something reasonable.

Ways to handle missing values:

1. **Remove** the sample (drop the row), or drop a feature that is missing almost everywhere.
2. **Impute a statistic** of that feature: the mean, the median, or the most frequent value.
3. **Impute a sensible value from the application**, chosen with domain knowledge (the lecture’s income example fills blanks with 50 because that value is reasonable for the problem, not because a formula produced it).

Mean imputation is pulled by outliers. Median imputation stays at the middle of the sorted values. Most-frequent imputation is the natural choice for a categorical feature, where mean and median do not apply.

### Duplicates

- Delete the older record, or
- merge the duplicate records into one.

“Sidney” and “Sid” at the same address are duplicates only if you know they are the same person. That decision is domain knowledge.

### Invalid data

- Look the correct value up in an external source, or
- apply reasoning and domain knowledge (the lecture turns the impossible zip `8033A` into `80331`).

### Noise

Filter out the noise component. Do this carefully: a filter can remove part of the real signal along with the garbage characters or the grainy image.

### Outliers

- Remove them when they are **not** the focus of the analysis.
- Keep them and analyze them more closely when they **are** the focus, such as fraud detection.

| Problem | Fix the lectures name |
|---|---|
| Missing | Drop the row/feature, or impute mean, median, most frequent value, or a domain value |
| Duplicate | Delete the older record, or merge |
| Invalid | External source, or domain reasoning |
| Noise | Filter, without wiping out the real pattern |
| Outlier | Remove if it is a nuisance; investigate if it is the subject |

### Spark

```python
from pyspark.ml.feature import Imputer

clean = df.na.drop(subset=["age"])                       # remove rows missing age
clean = df.dropDuplicates(["name", "address"])

imputer = Imputer(
    inputCols=["income"], outputCols=["income"], strategy="median"
)   # strategy can be "mean", "median", or "mode" (most frequent)
clean = imputer.fit(df).transform(df)

clean = clean.na.fill({"city": "Unknown"})               # a chosen stand-in
```

---

## 13. Feature selection

**Feature selection** (also called **feature engineering**) means choosing the smallest set of features that still captures the problem. There is a tradeoff between **expressiveness** (enough information) and **size** (fewer columns).

Domain knowledge decides what is relevant. The statistics from exploration help: correlation tells you which columns are redundant, and missing-value counts tell you which columns are mostly empty.

| Approach | What you do | Lecture example |
|---|---|---|
| Add | Derive a new feature from ones you have | `State` becomes an extra `In-State` flag (T/F) |
| Remove | Drop a column | Drop features that are strongly correlated with another feature, features with many missing values, and irrelevant features such as an ID or a row number |
| Combine | Collapse several columns into one | Height and weight become BMI |
| Recode | Change how a feature is represented | **Discretization:** age in years becomes {teenager, young adult, adult, senior} |
| Break up | Split one field into several | One address string becomes street, state, and zip |

An ID is irrelevant because it does not describe a general pattern. It is different for every row.

**Exam point.** Feature selection changes **which columns exist**. Feature transformation (next section) changes **the values** in those columns. Combining features is taught in this course under selection. Scaling is taught under transformation.

### Spark

```python
from pyspark.sql.functions import col, when
from pyspark.ml.feature import Bucketizer, VectorAssembler

df = df.drop("id")
df = df.withColumn("bmi", df.weight / (df.height ** 2))
df = df.withColumn("in_state", when(col("state") == "CA", 1).otherwise(0))

# discretization of age: 0-12, 13-17, 18-64, 65+
bins = Bucketizer(splits=[0, 13, 18, 65, 200], inputCol="age", outputCol="age_group")

# the columns the model will actually see
assembler = VectorAssembler(
    inputCols=["bmi", "in_state", "age_group"], outputCol="features"
)
```

Spark’s learning algorithms expect one vector column named something like `features`, not a list of separate numeric columns. `VectorAssembler` builds that vector. That is the usual last step of feature selection in Spark.

---

## 14. Feature transformation

**Feature transformation** maps feature values to a new set of values, so the data is in a form that the analysis can use.

The caveat from the lectures: do not filter or compress away the characteristics that matter.

| Operation | What it does | Lecture example |
|---|---|---|
| Scaling to a range | Stretch or squeeze a column into a fixed interval, often **[0, 1]** | Income (0 to 100,000) and years of employment (0 to 30) both mapped onto [0, 1] |
| Standardization (zero-normalization) | Rescale so the column has **mean 0** and **standard deviation 1** | Same idea, different target scale |
| Filtering | Remove a noise component | Remove grain from an image, or noise from an audio signal |
| Aggregation | Combine many fine-grained values into a coarser one | Average wind speed every 10 minutes becomes average wind speed every hour |
| Dimensionality reduction | Replace many columns with fewer | Covered in the next section (PCA) |

### When scaling matters

Scale features when their **raw ranges are very different** and the method uses distances or variances. Income in the tens of thousands will otherwise swamp years-of-employment in the tens. After both sit on [0, 1], or both have standard deviation 1, they contribute on comparable terms.

This is why the clustering lecture tells you to normalize inputs before you measure similarity. It is also the right preparation before PCA, because PCA follows variance: an unscaled column with a huge range becomes the first component just because of its units.

Linear regression can be fit on the original units. Scaling is still useful there when you want to **compare coefficients** with each other.

Two scalers, two jobs:

- **Min–max scaling** puts each feature on a fixed range such as [0, 1].
- **Standardization / zero-normalization** puts each feature at mean 0 and standard deviation 1. It does not force a hard minimum and maximum.

### Spark

```python
from pyspark.ml.feature import MinMaxScaler, StandardScaler
from pyspark.sql.functions import window, avg

# values scaled to [0, 1]
MinMaxScaler(inputCol="features", outputCol="scaled")

# zero-normalization: mean 0 and standard deviation 1
# withMean must be True; the default only divides by the standard deviation
StandardScaler(inputCol="features", outputCol="scaled", withMean=True, withStd=True)

# aggregation: 10-minute readings -> hourly average (the wind-speed example)
df.groupBy(window("timestamp", "1 hour")).agg(avg("wind").alias("avg_wind"))
```

Fit the scaler on the **training** data only, then apply that same fitted scaler to validation and test data. Fitting it on all rows lets information from the test set influence the model. That is the same rule as the validation lecture: the test set stays out of model creation.

---

## 15. Dimensionality reduction and PCA

The **dimensionality** of a table is the number of features (the number of axes). Two features is 2D data, three features is 3D data, and a real dataset may have hundreds.

**Curse of dimensionality.** As the number of features grows, the data becomes increasingly **sparse**: samples sit far apart in the space. Analysis results degrade.

**Dimensionality reduction** replaces the original features with a smaller set. Keep the dimensions that carry information. Drop the ones that do not.

Benefits you should be able to state:

- fewer dimensions, so the data is less sparse
- irrelevant dimensions are gone
- the later model is simpler and the degraded-results problem is reduced

### Principal component analysis (PCA)

PCA builds a **new coordinate system** for the same points.

- **PC1** is the direction of greatest variance.
- **PC2** is the direction of the next greatest variance, perpendicular to PC1.
- Later components follow the same way, each capturing the most remaining variance.

The first few components usually hold most of the variance, so the data can live in that lower-dimensional space (the lecture’s picture keeps PC1 and drops PC2).

The cost is interpretation. The original axes had names: income, age, occupation. The new axes are PC1, PC2, PC3. Each one is a mixture of the original features, so you cannot read a component as “this axis is income”.

| | Original features | Principal components |
|---|---|---|
| Meaning | A real measurement | A variance-ordered mix of measurements |
| How many you keep | All of them | The first few, which hold most of the variance |
| Easy to explain? | Yes | Harder |

PCA is unsupervised. It uses the feature values only. It does not look at a class label.

**Exam point.** PC1 is “maximum variance”, not “the most important feature by name”. Scaling first stops a wide-range column from winning that contest on units alone.

### Spark

```python
from pyspark.ml.feature import PCA

pca = PCA(k=2, inputCol="scaled", outputCol="pcs")  # k = how many components to keep
pca_model = pca.fit(train)
print(pca_model.explainedVariance)   # share of variance in PC1, PC2, ...
train_2d = pca_model.transform(train)
```

`explainedVariance` tells you how much of the spread each kept component captures. If the first two entries already sum to nearly 1, two dimensions carry almost all of the variance.

---

## 16. Classification

**Classification** predicts a **category** from the input variables. The target is categorical. The task is **supervised**: each training sample has a known label.

The lectures’ weather table:

| Temperature | Humidity | Wind speed | Weather (target) |
|---|---|---|---|
| 79 | 48 | 2.7 | Sunny |
| 60 | 80 | 3.8 | Rainy |
| 68 | 45 | 17.9 | Windy |
| 57 | 77 | 4.2 | Cloudy |

Names for the target column: target variable, class variable, class label, class category.

| | Binary (binomial) | Multi-class (multinomial) |
|---|---|---|
| How many target values | Exactly two | More than two |
| Examples | Rain tomorrow or not. Legitimate or fraudulent transaction | Which product the customer will buy. Tweet is positive, negative, or neutral. Loan is low, medium, or high risk |

Building a classifier means finding **decision boundaries**: the borders in the input space that separate one class from another.

**Exam point.** The learning objectives say “binomial” and “multinomial”. The slides also say “binary” and “multi-class”. Use either pair. Both mean “two classes” versus “more than two classes”.

### Spark

```python
from pyspark.ml.feature import StringIndexer, VectorAssembler
from pyspark.ml.classification import DecisionTreeClassifier

df = StringIndexer(inputCol="weather", outputCol="label").fit(df).transform(df)
df = VectorAssembler(
    inputCols=["temperature", "humidity", "wind"], outputCol="features"
).transform(df)

tree = DecisionTreeClassifier(labelCol="label", featuresCol="features")
```

Two classes or many classes is the same class of estimator in Spark. The label indexer assigns one number per category, and the tree predicts that number.

---

## 17. Building a model and applying it

A **machine learning model** is a mathematical function with **parameters** that maps inputs to an output.

```text
y = f(x; parameters)
```

The lecture’s picture is the straight line `y = mx + b`.

- `x` is the input, `y` is the output.
- `m` and `b` are the parameters (slope and intercept).
- Changing the parameters changes the mapping. With `m = 2` and `b = -1`, the input `x = 1` gives `y = 1`. With `m = 2` and `b = +1`, the same input gives `y = 3`.

**Building** (training) means adjusting those parameters so the mapping fits the training data. A **learning algorithm** does the adjusting.

**Applying** (testing) means freezing the parameters and computing outputs for new data.

| | Training phase | Testing phase |
|---|---|---|
| What happens | The learning algorithm adjusts parameters | The finished model is applied |
| Which data | Training data | New data the model did not learn from |
| What comes out | A fitted model | Predictions |

Parameters have to be adjusted because a model with the wrong slope, the wrong intercept, or the wrong decision boundary maps inputs to the wrong outputs. Training is the search for parameters that match the examples you already know.

For a classifier, the fitted parameters define the decision boundaries. For linear regression, the fitted parameters are the slope(s) and the intercept.

### Spark

`fit` builds. `transform` applies.

```python
model = tree.fit(train)                         # training: adjust parameters
predictions = model.transform(test)             # testing: apply to new rows
predictions.select("label", "prediction").show()
```

---

## 18. Generalization, overfitting, underfitting

For a classifier:

- **Success:** predicted label equals the true label.
- **Error:** predicted label differs from the true label.
- **Error rate** (misclassification error) = (number of errors) / (number of samples).

| Error | Measured on | What it tells you |
|---|---|---|
| Training error | The data used to adjust parameters | How well the model memorized the training samples |
| Test error | Data held out from training | How well the model will do on new data |

**Generalization** means the model performs well on new data. The **test error is the generalization error**.

**Overfitting** means the model fits **noise** in the training data, not just the real structure. The usual cause in this course is an **overly complex** model (a decision tree with too many nodes is the lecture’s example). Symptoms:

- training error looks very small
- test error (generalization error) is much larger

Overfitting causes **poor generalization**. Avoid it because the model will be wrong on the new cases you actually care about, even though it looks excellent on the data it trained on.

**Underfitting** is the opposite problem: the model is too simple and misses the structure. Both training error and test error stay high.

The useful model sits between them: complex enough to capture the structure, simple enough that it does not chase noise.

```text
Underfitting          Just right           Overfitting
too simple            captures structure   fits noise
high train error      both errors modest   tiny train error
high test error       test error close     test error much higher
```

**Exam point.** A training accuracy of 99% does not mean the model is good. Ask for the test error. A large gap, with training error far below test error, is the overfitting pattern.

### Spark

Compare the two errors. A large gap is the warning.

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator
acc = MulticlassClassificationEvaluator(labelCol="label", metricName="accuracy")
print("train", acc.evaluate(model.transform(train)))
print("test ", acc.evaluate(model.transform(test)))
```

---

## 19. Decision trees and pruning

A **decision tree** classifies by asking a sequence of questions about the features. Each internal node is a question. Each leaf is a predicted class.

**Induction** means growing that tree from the training data.

A tree overfits when its nodes fit noise. A fully grown tree can isolate individual odd training rows. It then fails on new rows. The fix is to **limit the number of nodes**, which limits the complexity of the tree.

| | Pre-pruning | Post-pruning |
|---|---|---|
| When | Stop **while growing**, before the tree is fully grown | Grow to the **maximum**, then cut back |
| How | Restrictive stopping rules | Remove nodes from the **bottom up**. Replace a subtree with a leaf if generalization error improves or stays the same |
| Stopping / cutting rules | Stop if the number of records in a node is below a threshold. Stop if the improvement in the **impurity** measure is below a threshold | Keep a simpler leaf whenever the more complicated subtree does not improve generalization |
| Use in practice | Available | **Used more often** |
| Cost | Cheaper | **More expensive**, because the full tree is built first |

**Impurity** is how mixed the class labels are inside a node. A node that contains only one class is pure. Splits are chosen to reduce impurity. Pre-pruning gives up on a split when the impurity improvement is tiny, because that split is likely fitting noise.

Spark grows the tree with pre-pruning controls. It does not grow a full tree and then post-prune. Map the lecture’s two stopping rules onto Spark parameters:

| Lecture rule | Spark parameter |
|---|---|
| Stop if the node has too few records | `minInstancesPerNode` |
| Stop if impurity barely improves | `minInfoGain` |
| Limit how complex the tree can get | `maxDepth` |

Spark’s impurity measure is **Gini** (the default) or **entropy**.

### Spark

```python
from pyspark.ml.classification import DecisionTreeClassifier

tree = DecisionTreeClassifier(
    labelCol="label",
    featuresCol="features",
    impurity="gini",            # or "entropy"
    maxDepth=5,                 # cap complexity
    minInstancesPerNode=10,     # stop if the node is smaller than this
    minInfoGain=0.01,           # stop if impurity barely improves
)
model = tree.fit(train)
```

---

## 20. Training, validation, and test sets

Overfitting grows as the model gets more complex (more nodes, more training). Training error keeps falling. At some point extra complexity only fits noise, and error on unseen data rises. You need a signal for **when to stop**.

That signal is a **validation set**: data that is held out from parameter fitting and checked during training.

| Set | Used for | May it influence the model? |
|---|---|---|
| Training | Adjust the parameters | Yes. This is the only data the learning algorithm fits |
| Validation | Decide when to stop, compare settings, estimate generalization while you are still building | Yes, but only for those choices. It is not used to fit the parameters |
| Test | A final estimate of performance on new data | **No. It cannot be used in any way during model creation** |

The lecture’s curve: as the number of nodes grows, training error falls. Validation error falls and then rises. **Stop at the node count where validation error is lowest.** Past that point the model is overfitting.

If you pick that stopping point by looking at the **test** set, the test error is no longer an honest report of new-data performance. You have used it to build.

### Four ways to build the validation set

The learning objective says “three ways”. The slides list four. The second is a repeated form of the first. Know all four names, including **random subsampling**, which is the objective’s name for repeated holdout.

| Method | What happens | What you report |
|---|---|---|
| Holdout | Split once. Train on one part. Use the held-out part to decide when to stop | The validation error of that one split |
| Repeated holdout (random subsampling) | Repeat the holdout several times. Each time, choose a **different** random holdout set | The **average** of the validation errors |
| k-fold cross-validation | Split the data into k blocks. Do k rounds. In round i, block i is the validation set and the other k−1 blocks are the training set | The average validation error across the k rounds |
| Leave-one-out cross-validation | k-fold with k = N, the number of samples. Each round holds out **one** sample | The average over all N rounds |

k-fold uses every sample as a validation sample exactly once per full run. Leave-one-out trains on almost all of the data every round, and it costs N separate trainings. One holdout split can be lucky or unlucky; averaging over repetitions or folds steadies the estimate.

Uses of a validation set, from the lectures:

- avoid overfitting (decide when to stop)
- estimate generalization performance while you can still change the model

Both validation and test estimate generalization. Only the test estimate is the final one, because the validation set has already been used to make choices.

### Spark

One holdout split into three sets:

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

train, val, test = df.randomSplit([0.6, 0.2, 0.2], seed=42)
acc = MulticlassClassificationEvaluator(labelCol="label", metricName="accuracy")
model = tree.fit(train)
print("val ", acc.evaluate(model.transform(val)))    # used to choose depth
print("test", acc.evaluate(model.transform(test)))   # touched once, at the end
```

k-fold, searching over tree depth. The test set stays outside `fit`:

```python
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

grid = ParamGridBuilder().addGrid(tree.maxDepth, [3, 5, 7]).build()
cv = CrossValidator(
    estimator=tree,
    estimatorParamMaps=grid,
    evaluator=MulticlassClassificationEvaluator(labelCol="label", metricName="accuracy"),
    numFolds=5,
)
best = cv.fit(train)             # five-fold validation happens inside train
best.transform(test)             # test was never part of the search
```

---

## 21. Confusion matrix

A **confusion matrix** counts how a classifier’s predictions line up with the true labels. For a binary problem, pick one class as the **positive** class (the lecture uses “Yes, this animal is a mammal”).

|  | Predicted Yes | Predicted No |
|---|---|---|
| **True Yes** | True positive (TP) | False negative (FN) |
| **True No** | False positive (FP) | True negative (TN) |

Rows are the **true** label. Columns are the **predicted** label. Memorize that orientation, because swapping it swaps FP and FN.

| Cell | In words |
|---|---|
| TP | Truth is yes, prediction is yes. Correct accept |
| TN | Truth is no, prediction is no. Correct reject |
| FP | Truth is no, prediction is yes. False alarm |
| FN | Truth is yes, prediction is no. A miss |

The lecture’s reading of the off-diagonal cells:

- A high **FN** means classifying the **positive** class is problematic (true positives are being called negative).
- A high **FP** means classifying the **negative** class is problematic (true negatives are being called positive).

### Worked example

Ten animals. The matrix from the lecture:

|  | Predicted Yes | Predicted No |
|---|---|---|
| True Yes | TP = 3 | FN = 2 |
| True No | FP = 1 | TN = 4 |

- Correct predictions = TP + TN = 7, out of 10.
- Incorrect predictions = FP + FN = 3, out of 10.

```text
Accuracy = (TP + TN) / (TP + TN + FP + FN) = (3 + 4) / 10 = 0.7
Error    = (FP + FN) / (TP + TN + FP + FN) = 3 / 10 = 0.3
Error    = 1 − accuracy
```

**Exam point.** Be able to fill the four cells from a list of (true, predicted) pairs, then compute accuracy from the cells. Check that the four cells sum to the number of samples.

### Spark

```python
preds.groupBy("label", "prediction").count().show()   # this is the matrix, in long form
preds.crosstab("label", "prediction").show()          # same counts as a table
```

The numeric labels come from `StringIndexer`, so 0.0 and 1.0 stand for the original class names. `indexer.labels` tells you which name is which index.

---

## 22. Accuracy, precision, recall, F-measure

These all score a classifier. Accuracy and error look at every sample. Precision, recall, and the F-measure look at the positive class, which is why they survive class imbalance.

| Metric | Formula | Plain meaning | Lecture’s word |
|---|---|---|---|
| Accuracy | (TP + TN) / (TP + TN + FP + FN) | Fraction correct | — |
| Error rate | (FP + FN) / (TP + TN + FP + FN) = 1 − accuracy | Fraction wrong | Misclassification error |
| Precision | TP / (TP + FP) | Of the samples **predicted** positive, how many truly are | Exactness |
| Recall | TP / (TP + FN) | Of the samples that **truly are** positive, how many you found | Completeness |
| F1 | 2 · precision · recall / (precision + recall) | Balance of precision and recall | Evenly weighted F |

From the mammal matrix (TP 3, FN 2, FP 1, TN 4):

```text
Precision = 3 / (3 + 1) = 0.75
Recall    = 3 / (3 + 2) = 0.60
F1        = 2 · 0.75 · 0.60 / (0.75 + 0.60) = 0.90 / 1.35 = 2/3 ≈ 0.667
```

Precision uses the column of positive predictions (TP + FP). Recall uses the row of true positives (TP + FN).

### Why accuracy misleads

**Class imbalance:** one class is rare. The lecture’s tumor example: 3% of samples are cancer, 97% are not. A model that **always predicts “not cancer”** has accuracy 97% and detects **no** cancer cases.

On 1000 patients (30 cancer, 970 not), that model has TP = 0, FP = 0, TN = 970, FN = 30.

- Accuracy = 970 / 1000 = 0.97
- Recall = 0 / 30 = 0

The rare class, which is the class you care about, is completely missed. Accuracy looks excellent because the huge true-negative count dominates the fraction. Precision and recall focus on the positive class, so they expose this. The goal is to **maximize precision and recall together**, not accuracy alone.

When the model never predicts the positive class, precision divides by zero (no positive predictions). Recall is defined and equals 0.

### F-measure weights

The general form is

```text
Fβ = (1 + β²) · precision · recall / (β² · precision + recall)
```

| Score | β | What is weighted more |
|---|---|---|
| F1 | 1 | Precision and recall equally |
| F2 | 2 | Recall |
| F0.5 | 0.5 | Precision |

β above 1 favors recall (finding all the positive cases). β below 1 favors precision (keeping the positive predictions pure).

**Exam point.** Given TP, FP, FN, TN, compute all five numbers. Given a story about a rare dangerous class, explain why accuracy is the wrong headline and recall is the number that shows the failure.

### Spark

```python
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

for metric in ["accuracy", "weightedPrecision", "weightedRecall", "f1"]:
    score = MulticlassClassificationEvaluator(
        labelCol="label", predictionCol="prediction", metricName=metric
    ).evaluate(preds)
    print(metric, score)
```

`f1` here is F1. Spark’s weighted precision and recall average the per-class scores by how common each class is, which is the right reading when there are more than two classes.

---

## 23. Regression

**Regression** predicts a **number** from the input variables. The target is numeric. The task is **supervised**.

| | Classification | Regression |
|---|---|---|
| Output | A category | A number |
| Target type | Categorical | Numeric |
| Supervised? | Yes | Yes |
| Lecture examples | Tumor type, rain or not, fraud or not, sentiment | Tomorrow’s high temperature, house price, product demand, power usage, amount of rain, a numeric likelihood that a drug works |
| Typical error | Misclassification rate | How far the predicted number is from the true number (the residual) |
| Decision boundaries? | Yes | No. The model produces a numeric prediction |

Training and testing mean the same thing as in classification. The training / validation / test split means the same thing: fit parameters on training data, use validation to stop before overfitting, touch the test set once at the end.

Accuracy, precision, recall, and the confusion matrix do **not** apply, because there is no class label to be right or wrong about. A prediction of 71 when the truth is 70 is a small error, not a “false positive”.

### Spark

```python
from pyspark.ml.regression import LinearRegression
from pyspark.ml.evaluation import RegressionEvaluator

lr = LinearRegression(featuresCol="features", labelCol="temp_tomorrow")
model = lr.fit(train)
preds = model.transform(test)

rmse = RegressionEvaluator(
    labelCol="temp_tomorrow", predictionCol="prediction", metricName="rmse"
).evaluate(preds)
```

---

## 24. Linear regression and least squares

**Linear regression** models the relationship between a numeric target and the inputs as a **straight line** (or a flat plane, when there are several inputs).

The lecture’s task: given petal width, predict petal length. After fitting, a petal width of 1.5 predicts a petal length of 4.5. Training is what discovers the line that makes that mapping.

### Simple vs multiple

| | Simple linear regression | Multiple linear regression |
|---|---|---|
| Inputs | One variable | More than one variable |
| Equation | `y = mx + b` | `y = b0 + b1 x1 + b2 x2 + … + bp xp` |
| Parameters | Slope `m`, intercept `b` | Intercept `b0`, one slope per input |
| Output | One number | One number |

Both predict a **single** numeric target. “Multiple” means multiple inputs, not multiple outputs.

`b` or `b0` is the prediction when all inputs are 0. Each slope is how much the prediction changes when that input increases by one unit.

### Least squares

A **residual** is the distance from a sample to the regression line: actual value minus predicted value.

**Least squares** chooses the parameters that make the **sum of squared residuals** as small as possible.

```text
minimize   Σ (y_i − predicted_y_i)²
```

Squaring does two jobs: a point below the line and a point above it both count as error, and large misses count more than small ones. The fitted line is the one with the smallest total of those squared distances. That is what `LinearRegression` fits in Spark.

Applying the model is substitution. Once `m` and `b` are fixed, a new `x` produces `y = mx + b`.

### Spark

```python
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.regression import LinearRegression

# simple: one input column. multiple: add more names to inputCols
data = VectorAssembler(inputCols=["petal_width"], outputCol="features").transform(df)
model = LinearRegression(featuresCol="features", labelCol="petal_length").fit(train)

print(model.coefficients)   # slopes (m, or b1, b2, ...)
print(model.intercept)      # b
print(model.summary.rootMeanSquaredError)
```

RMSE is the square root of the average squared residual, so it is the least-squares error expressed back in the units of `y` (petal length, degrees, price). `metricName="r2"` on `RegressionEvaluator` is the fraction of variance the line explains: nearer to 1 is a tighter fit.

---

## 25. Cluster analysis

**Cluster analysis** organizes similar items into groups. It is **unsupervised**. Samples in the same cluster should be similar. Samples in different clusters should be different.

- **Intra-cluster** differences are minimized (tight groups).
- **Inter-cluster** differences are maximized (groups far apart from each other).

There is **no single correct clustering**. Clusters do not come with labels. You have to interpret them.

### Measuring “similar”

The lectures name three measures.

| Measure | Kind | What is small or large |
|---|---|---|
| Euclidean distance | Distance | Straight-line distance. Smaller means more alike |
| Manhattan distance | Distance | Sum of absolute differences along each axis (city-block distance). Smaller means more alike |
| Cosine similarity | Similarity | Cosine of the angle between two samples. Larger means more alike. 1 means the same direction |

**Normalize the inputs first.** If weight is in the tens and height is in the hundreds, Euclidean and Manhattan distances are dominated by height, and the clustering is a clustering of height alone. Scaling (previous section) puts the features on comparable ranges.

### What cluster results are used for

1. **Segmentation.** Study each group separately. Customer segments, news topics (science fiction, non-fiction, children’s), weather regimes.
2. **A category for a new sample.** Assign the new sample to the closest cluster and use that cluster as its label.
3. **Labels for a later classifier.** Once you have named the clusters, the cluster id can be the target of a supervised model.
4. **Anomaly detection.** A sample that sits far from every cluster, or in a tiny cluster of its own, is an anomaly worth a closer look.

Examples to be able to list: customer segments, weather patterns, groups of news articles, crime hot spots, land cover (desert, grass, …), tissue types in a medical image.

**Exam point.** “Is clustering supervised?” No. “Do clusters come with names?” No. “Is the clustering with the lower error automatically the true one?” No. Interpretation is part of the method, not a cleanup step after it.

### Spark

Scale, then cluster. The cluster id of each row is the segmentation.

```python
from pyspark.ml.feature import StandardScaler
from pyspark.ml.clustering import KMeans

scaled = StandardScaler(
    inputCol="features", outputCol="scaled", withMean=True, withStd=True
).fit(df).transform(df)

kmeans = KMeans(k=3, featuresCol="scaled", predictionCol="cluster")
model = kmeans.fit(scaled)
model.transform(scaled).select("cluster").show()
```

Spark’s `KMeans` uses Euclidean distance. Scaling is not optional if the columns have different units.

---

## 26. k-means

**k-means** is the course’s clustering algorithm. It is simple, efficient, and you must choose **k** yourself.

**k** is the number of clusters.

A **centroid** is the center of a cluster. In k-means it is the **mean** of the samples currently assigned to that cluster. There is one centroid per cluster, so there are k centroids.

### The algorithm

1. Choose **k initial centroids**.
2. Repeat:
   - assign every sample to the **closest** centroid
   - recompute each centroid as the **mean** of the samples assigned to it
3. Stop when a stopping criterion is reached.

Stopping criteria from the lectures:

- the centroids **stop changing**, or
- the number of samples that **change clusters** falls below a threshold

Spark also takes `maxIter`, a maximum number of repeats, so the loop cannot run forever.

One pass of “assign, then recompute the mean” is one iteration. The lecture’s pictures (initial centroids, assign, move the centroids, assign again, move again) are exactly this loop.

### Starting points matter

The final clusters are **sensitive to the initial centroids**. Two different starts can produce two different groupings.

The fix: run k-means several times with different random initial centroids, and keep the best result.

### Within-cluster sum of squared error (WSSE)

For one sample, the error is its distance to its centroid. The squared error is that distance squared. **WSSE** sums the squared errors of every sample in a cluster, then sums across all clusters.

- Comparing two runs with the **same k**, the smaller WSSE is the tighter clustering. “Better numerically” does not mean “the true grouping”.
- A **larger k always reduces WSSE**. At the extreme, each sample is its own cluster and WSSE is 0. So you cannot pick k by taking the smallest WSSE.

### Choosing k

| Approach | How |
|---|---|
| Visualization | Look at the points (when you can draw them) and see how many groups appear |
| Application-dependent | The problem dictates the number. The business wants four customer segments |
| Data-driven | Use the data to suggest k. The lecture’s method is the **elbow method** |

**Elbow method.** Plot WSSE against k. WSSE drops quickly at first, then flattens. The bend, the elbow, is the suggested k. In the lecture’s sketch the elbow is at **k = 3**: moving from 2 to 3 helps a lot, and moving past 3 helps little.

### Interpreting the result

Compare the centroids. A centroid is a point in the original feature space, so its coordinates say what that group is like (older and higher income, versus younger and lower income). You name the clusters after that comparison. The algorithm will not name them for you.

| Strength | Limitation |
|---|---|
| Simple to understand and implement | You must choose k |
| Efficient | The result depends on the initial centroids |
| WSSE gives a numeric way to compare runs of the same k | Lower WSSE from a larger k is not automatically a better clustering |

### Spark

```python
from pyspark.ml.clustering import KMeans

# several random starts; keep the lowest WSSE. k stays fixed here.
best, best_cost = None, None
for seed in range(5):
    m = KMeans(k=3, seed=seed, featuresCol="scaled").fit(data)
    if best is None or m.summary.trainingCost < best_cost:
        best, best_cost = m, m.summary.trainingCost

print(best.clusterCenters())       # the k centroids: interpret these
print(best_cost)                   # WSSE of the winning start

# elbow: WSSE against k. Look for the bend. Do not pick the minimum.
for k in range(2, 8):
    cost = KMeans(k=k, seed=1, featuresCol="scaled").fit(data).summary.trainingCost
    print(k, cost)
```

`summary.trainingCost` is Spark’s name for the WSSE: the sum of squared distances from each point to its centroid.

---

## 27. Association analysis

**Association analysis** finds rules that capture associations between items. It is **unsupervised**. The classic setting is a set of baskets (transactions), each basket a set of items.

An **itemset** is a set of items, such as `{bread, milk}` or `{beer, diaper}`.

A **rule** has an antecedent and a consequent:

```text
X  =>  Y
{bread, milk}  =>  {diapers}
```

Read it as “if a basket contains X, then it also contains Y”. X is the **antecedent**. Y is the **consequent**.

### The three steps

1. **Create itemsets** from the transactions (singles, pairs, triples, …).
2. **Keep the frequent itemsets**, using support.
3. **Generate rules** from those frequent itemsets, and keep the rules that pass a confidence threshold.

### Support and confidence

```text
support(X) = (number of transactions that contain every item in X)
           / (number of transactions)

confidence(X => Y) = support(X ∪ Y) / support(X)
```

- Support measures how **common** an itemset is.
- Confidence measures how **reliable** the implication is: of the baskets that contain the antecedent, what fraction also contain the consequent.
- The denominator is the support of the **antecedent only**, not the consequent, and not the whole itemset.

You choose two thresholds:

- **Minimum support** drops rare itemsets before rules are built.
- **Minimum confidence** drops weak rules after they are built.

A k-itemset can form **2^k − 2** candidate rules. Each item can sit on the left or the right, which is 2^k assignments, minus the two useless ones (everything on the left, or everything on the right). A pair gives 2 rules. A triple gives 6. The two directions of the same pair have **different** confidences, so direction matters.

### Worked example

Five transactions. Minimum support = 3/5. Minimum confidence = 0.95.

| ID | Items |
|---|---|
| 1 | diaper, bread, milk |
| 2 | bread, diaper, beer, eggs |
| 3 | milk, diaper, beer, butter |
| 4 | bread, milk, diaper, beer |
| 5 | bread, milk, diaper, butter |

Support of the singles:

| Itemset | Support | Keep? (minimum 3/5) |
|---|---|---|
| {diaper} | 5/5 | yes |
| {bread} | 4/5 | yes |
| {milk} | 4/5 | yes |
| {beer} | 3/5 | yes |
| {butter} | 2/5 | no |
| {eggs} | 1/5 | no |

Butter and eggs are dropped. Any larger itemset that contains them is dropped too, because a set cannot be frequent if one of its pieces is infrequent. That pruning is the **Apriori** idea, and it is why the lectures build itemsets level by level instead of listing every possible subset.

Frequent itemsets at 3/5:

| Size | Itemset | Support |
|---|---|---|
| 1 | {bread} | 4/5 |
| 1 | {milk} | 4/5 |
| 1 | {beer} | 3/5 |
| 1 | {diaper} | 5/5 |
| 2 | {bread, milk} | 3/5 |
| 2 | {bread, diaper} | 4/5 |
| 2 | {milk, diaper} | 4/5 |
| 2 | {beer, diaper} | 3/5 |
| 3 | {bread, milk, diaper} | 3/5 |

`{bread, beer}` and `{milk, beer}` are 2/5, so they are not frequent. One summary frame in the lecture lists the 1-item sets without `{beer}`. Beer’s own support on the earlier frame is 3/5, which meets the minimum, and `{beer, diaper}` is kept. On an exam, apply the threshold yourself and keep every itemset at or above it. If you keep a set, keep its subsets too.

Rules from `{beer, diaper}` (support 3/5). Same itemset, two directions:

```text
{beer}   => {diaper}   confidence = (3/5) / (3/5) = 1.00   keep
{diaper} => {beer}     confidence = (3/5) / (5/5) = 0.60   drop
```

Every basket with beer also has diapers. The reverse is weaker, because diapers are in every basket and beer is only in three of them.

Rules the lecture computes from the triple `{bread, milk, diaper}`:

```text
{bread, milk}   => {diaper}   confidence = (3/5) / (3/5) = 1.00   keep
{bread, diaper} => {milk}     confidence = (3/5) / (4/5) = 0.75   drop
```

In this dataset, every basket that contains bread and milk also contains diapers.

The other four rules from that triple, so you can see all 2^3 − 2 = 6:

```text
{milk, diaper} => {bread}         (3/5) / (4/5) = 0.75
{bread}  => {milk, diaper}        (3/5) / (4/5) = 0.75
{milk}   => {bread, diaper}       (3/5) / (4/5) = 0.75
{diaper} => {bread, milk}         (3/5) / (5/5) = 0.60
```

Only `{bread, milk} => {diaper}` clears a minimum confidence of 0.95.

### Algorithms named in the lectures

All three do the same job: make itemset creation and rule generation efficient. They differ in how they count.

| Algorithm | Idea, in one sentence | In Spark? |
|---|---|---|
| Apriori | Build candidates level by level (size 1, then 2, then 3) and drop any candidate that contains an infrequent subset | Named in the lecture |
| FP-Growth | Compress the baskets into a frequent-pattern tree and mine that tree, without listing every candidate | This is the algorithm Spark implements |
| Eclat | Store the transaction ids of each item and intersect those lists to count larger itemsets | Named in the lecture |

### Using the rules

The rules are easy to read, and that is part of their appeal. Usefulness is still **subjective**. A rule can clear both thresholds and be useless to the business (a consequent that almost everyone buys will produce high confidence and a dull recommendation). Domain knowledge decides which rules to act on: put those items on sale together, recommend the consequent, or look at treatments that co-occur.

Applications: items often purchased together, recommendations from purchase or browsing history, web pages accessed together, treatments that go together.

**Exam point.** Given a handful of transactions, compute support of an itemset and confidence of both directions of a rule. State which itemsets a minimum support removes, and which rules a minimum confidence removes. Remember that confidence divides by the antecedent.

### Spark

Spark’s implementation is FP-Growth. `minSupport` is a **fraction**, so 3 out of 5 is `0.6`, not `3`.

```python
from pyspark.sql.functions import split
from pyspark.ml.fpm import FPGrowth

baskets = spark.read.text("baskets.txt").select(split("value", ", ").alias("items"))
# each row is one transaction; items is an array of strings

fp = FPGrowth(itemsCol="items", minSupport=0.6, minConfidence=0.95)
model = fp.fit(baskets)
model.freqItemsets.show()       # itemsets that clear minimum support
model.associationRules.show()   # rules; read antecedent, consequent, confidence
```

The rules table also includes lift. This course scores rules with **support** and **confidence**.

---

## 28. Night-before sheet

### Definitions in one line

| Term | Definition |
|---|---|
| Machine learning | Learn from data, without explicit rules, to find patterns and support decisions |
| Supervised | Target is given (labeled data). Classification, regression |
| Unsupervised | Target is not given. Clustering, association |
| Sample | One row. Also instance, observation, record, example |
| Feature | One column. Also variable, attribute, field, dimension |
| Numeric feature | Quantitative. A number |
| Categorical feature | Qualitative, nominal. A label |
| Summary statistic | A number that describes a set of values |
| Imputation | Replace a missing value with a reasonable one |
| Feature selection | Smallest set of columns that still captures the problem |
| Feature transformation | Map values to new values (scale, filter, aggregate) |
| Curse of dimensionality | More features make the data sparse and the analysis worse |
| PCA | New axes ordered by variance. PC1 has the most |
| Classification | Predict a category. Supervised |
| Regression | Predict a number. Supervised |
| Training | Adjust parameters on training data |
| Generalization | Works on new data. Measured by test error |
| Overfitting | Fits noise because the model is too complex. Low train error, high test error |
| Underfitting | Too simple to capture the structure |
| Pre-pruning | Stop growing the tree early |
| Post-pruning | Grow fully, then replace subtrees with leaves from the bottom up |
| Validation set | Held out to decide when to stop. Not the final test |
| Test set | Held out completely. Never used to build the model |
| TP / TN | Correct positive / correct negative |
| FP / FN | False alarm / miss |
| Accuracy | (TP + TN) / all |
| Precision | TP / (TP + FP). Exactness |
| Recall | TP / (TP + FN). Completeness |
| F1 | Harmonic balance of precision and recall |
| Residual | Actual minus predicted, the distance to the regression line |
| Least squares | Choose the line that minimizes the sum of squared residuals |
| Centroid | Center of a cluster. In k-means, the mean of its members |
| k | Number of clusters |
| WSSE | Sum of squared distances from each point to its centroid |
| Itemset | A set of items |
| Support | Fraction of transactions that contain the itemset |
| Confidence | support(X ∪ Y) / support(X) for the rule X => Y |
| CRISP-DM | Standard data-mining process: business, data, preparation, modeling, evaluation, deployment |

### Formulas

```text
mean        = sum / n
median      = middle of the sorted list (average the two middle values if n is even)
range       = max − min
accuracy    = (TP + TN) / (TP + TN + FP + FN)
error       = 1 − accuracy
precision   = TP / (TP + FP)
recall      = TP / (TP + FN)
F1          = 2 · P · R / (P + R)
Fβ          = (1 + β²) · P · R / (β² · P + R)
simple LR   = y = mx + b
multiple LR = y = b0 + b1 x1 + ... + bp xp
least sq.   = minimize Σ (y − predicted)²
support(X)  = count(transactions containing X) / count(transactions)
conf(X=>Y)  = support(X ∪ Y) / support(X)
rules       = 2^k − 2 from a frequent k-itemset
```

### Pairs that exams contrast

| A | B |
|---|---|
| Supervised: labeled target | Unsupervised: no target |
| Classification: category | Regression: number |
| Binary: 2 classes | Multi-class: more than 2 |
| Training: fit parameters | Testing: apply the frozen model |
| Validation: stop and compare | Test: final score, never used to build |
| Overfitting: fits noise | Underfitting: misses the structure |
| Pre-pruning: stop early | Post-pruning: grow, then cut. Used more, costs more |
| Holdout: one split | k-fold: each block is validation once |
| Repeated holdout: average several random splits | Leave-one-out: hold out one sample, N times |
| Precision: of predicted positives, how many are right | Recall: of true positives, how many you found |
| Min–max scale: fixed range, often [0, 1] | Standardization: mean 0, standard deviation 1 |
| Feature selection: which columns | Feature transformation: which values |
| Simple regression: one input | Multiple regression: several inputs, still one output |
| Intra-cluster: tight inside | Inter-cluster: separated between |
| Euclidean / Manhattan: smaller means closer | Cosine: larger means more alike |
| Support: how common | Confidence: how reliable the direction of the rule is |
| KNIME: GUI, smaller data | Spark: code, distributed, large data |
| Mean: pulled by extremes | Median: the middle, stable |
| High FN: positive class is hard | High FP: negative class is hard |

### Order of the work

```text
business question
-> acquire and join
-> explore (statistics, plots, missing values, odd values)
-> clean (drop, impute, dedupe, filter)
-> select and transform features, scale
-> reduce dimensions if the table is wide and sparse
-> split train / validation / test
-> fit on train
-> use validation to stop or to choose k, depth, thresholds
-> evaluate once on test
-> report, deploy, monitor, repeat
```

### Numbers worth remembering from the lectures

- Ages: mean 51.1, median 46, modes 42 and 78, range 66, sd ≈ 23.4, skew ≈ 0.30
- Height and weight correlation ≈ 0.89
- Mammal matrix: TP 3, FN 2, FP 1, TN 4 → accuracy 0.7, error 0.3, precision 0.75, recall 0.60, F1 = 2/3
- Always-predict-not-cancer when 3% are cancer → accuracy 97%, recall 0
- Market basket, 5 transactions, minimum support 3/5, minimum confidence 0.95
- `{bread, milk} => {diaper}` has confidence 1 and is kept
- `{beer} => {diaper}` has confidence 1; the reverse has confidence 0.6
- Elbow sketch suggests k = 3
- Petal width 1.5 maps to petal length 4.5 on the lecture’s fitted line

---

## 29. Questions you should be able to answer

These follow the “after this video you will be able to” objectives in the lectures. If you can answer each one out loud, without looking up, you have covered the folder.

**Overview and process**

1. What is machine learning? Give three everyday applications.
2. Name the four techniques and the goal of each.
3. Which two are supervised, and what does “supervised” mean?
4. List the five steps, the goal of each, and two activities inside each.
5. Why is the process iterative?
6. What does CRISP-DM stand for? List the six phases and the goal of each.
7. Where does “decide the business success criteria” live, and where do you check the model against those criteria?
8. What is a KNIME node, port, and workflow?
9. What is Spark MLlib, which languages can call it, and how does it differ from KNIME?

**Data**

10. Define sample and feature, and give two other names for each.
11. How does a categorical feature differ from a numeric one? Give two examples of each.
12. Why explore data, and what three things are you looking for?
13. Name the two families of exploration techniques.
14. Classify mean, standard deviation, skewness, and correlation as location, spread, shape, or dependence.
15. From a list of ten numbers, compute mean, median, mode, and range.
16. What does a correlation of 0.89 mean? What does a correlation near 0 mean?
17. What does a cell of a contingency table contain?
18. Which plot would you draw for a distribution, a time trend, a relationship between two numbers, a category count, and a comparison of distributions?
19. What is the box in a box plot, and how do you see positive skew?
20. Why does preparation matter? What does “garbage in, garbage out” mean in this course?
21. Name three data-quality problems and a fix for each.
22. Define imputation, and list three ways to fill or remove a missing value.
23. When do you remove an outlier, and when do you study it?
24. What is the goal of feature selection? Give an example of adding, removing, combining, recoding, and splitting a feature.
25. What is discretization?
26. Name three feature transformations. When is scaling important, and what are the two scalers’ targets?
27. What is the curse of dimensionality?
28. What does PCA do to the coordinate system? Why are PC1 and PC2 harder to explain than “income” and “age”?

**Models and evaluation**

29. Define classification. How does binary differ from multi-class?
30. What is a model parameter? What is the difference between building a model and applying it?
31. Define generalization, training error, and test error.
32. What is overfitting, what causes it in this course, and why is it a problem? How does underfitting differ?
33. Define pre-pruning and post-pruning. Which is used more often, and which costs more?
34. What are the two pre-pruning stopping rules?
35. What is each of training, validation, and test data allowed to do?
36. Explain holdout, repeated holdout, k-fold, and leave-one-out.
37. Draw a confusion matrix and place TP, TN, FP, and FN. What does a large FN mean?
38. Compute accuracy, error, precision, recall, and F1 from TP, FP, FN, TN.
39. Why can 97% accuracy be a bad classifier? What does F2 weight more heavily than F0.5?
40. How does regression differ from classification? Give three regression applications.
41. Write the simple and multiple linear equations. What does least squares minimize?
42. Is clustering supervised? What do “intra-cluster” and “inter-cluster” mean?
43. Name three similarity measures and three uses of a finished clustering.
44. Why normalize features before clustering?
45. State the k-means steps. What do k and “centroid” mean? When does the loop stop?
46. Why run k-means more than once? Why is the smallest WSSE a bad way to choose k? What does the elbow show?
47. Define itemset, antecedent, consequent, support, and confidence.
48. From a few baskets, compute support and both directions of a rule, and apply a minimum support and a minimum confidence.
49. How many candidate rules does a frequent 3-itemset produce? Name Apriori, FP-Growth, and Eclat.
50. Why can a high-confidence rule still be useless?

**Spark syntax, at the level of a few lines**

51. Read a CSV into a DataFrame and show its schema.
52. Compute a mean, a correlation, and a contingency table.
53. Drop duplicate rows and impute a column with the median.
54. Build a `features` vector and scale it to mean 0 and standard deviation 1.
55. Fit a PCA with 2 components and read how much variance they explain.
56. Turn a string label into a numeric label and fit a decision tree.
57. Split train, validation, and test. Fit on train and evaluate on test.
58. Set the decision-tree parameters that implement pre-pruning.
59. Run a 5-fold cross-validation over `maxDepth`.
60. Fit k-means, read the centroids, and read the WSSE.
61. Fit FP-Growth with minimum support 0.6 and minimum confidence 0.95, and show the rules.
62. Save a model and load it to score new data.

---

*Covers the lecture PDFs in this folder: overview, categories, process, CRISP-DM, tools, terminology, exploration, summary statistics, plots, preparation, data quality, feature selection, feature transformation, dimensionality reduction, classification, model building, generalization, decision-tree pruning, validation, confusion matrix, evaluation metrics, regression, linear regression, cluster analysis, k-means, and association analysis.*
