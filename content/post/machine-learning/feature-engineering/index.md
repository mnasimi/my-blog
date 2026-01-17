---
title: "Feature Engineering: The Art of ML Data Preparation"
description: "Master the techniques that transform raw data into powerful model features."
slug: feature-engineering
date: 2024-05-30
categories:
  - Machine Learning
tags:
  - feature-engineering
  - data-science
  - preprocessing
  - machine-learning
---

Feature engineering is often the difference between a mediocre model and a great one. It's where domain knowledge meets data science.

## What Is Feature Engineering?

Feature engineering is the process of using domain knowledge to create features that make machine learning algorithms work better.

```
Raw Data ──▶ Feature Engineering ──▶ Better Features ──▶ Better Model
```

## Handling Missing Values

### Strategies

```python
import pandas as pd
import numpy as np

# Drop rows with missing values
df.dropna()

# Fill with mean/median/mode
df['age'].fillna(df['age'].median(), inplace=True)

# Fill with a constant
df['category'].fillna('Unknown', inplace=True)

# Forward/backward fill (time series)
df['value'].fillna(method='ffill', inplace=True)

# Create a "missing" indicator
df['age_missing'] = df['age'].isna().astype(int)
```

### Choosing a Strategy

| Data Type | Good Strategy |
|-----------|--------------|
| Numerical (MCAR) | Mean/Median |
| Numerical (MAR) | Model-based imputation |
| Categorical | Mode or "Missing" category |
| Time Series | Forward/backward fill |

## Encoding Categorical Variables

### One-Hot Encoding

```python
# For nominal categories
pd.get_dummies(df['color'], prefix='color')

# Result:
# color_red  color_blue  color_green
#    1           0            0
#    0           1            0
```

### Label Encoding

```python
from sklearn.preprocessing import LabelEncoder

# For ordinal categories
le = LabelEncoder()
df['size_encoded'] = le.fit_transform(df['size'])
# small=0, medium=1, large=2
```

### Target Encoding

```python
# Encode based on target variable mean
target_means = df.groupby('category')['target'].mean()
df['category_encoded'] = df['category'].map(target_means)
```

## Scaling and Normalization

### StandardScaler (Z-score)

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
df['age_scaled'] = scaler.fit_transform(df[['age']])
# Mean = 0, Std = 1
```

### MinMaxScaler

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
df['age_normalized'] = scaler.fit_transform(df[['age']])
# Range [0, 1]
```

### When to Use What

| Method | Use When |
|--------|----------|
| StandardScaler | Normal distribution, algorithms sensitive to magnitude |
| MinMaxScaler | Bounded data, neural networks |
| RobustScaler | Many outliers |
| Log transform | Skewed distributions |

## Creating New Features

### Mathematical Transformations

```python
# Ratios
df['price_per_sqft'] = df['price'] / df['sqft']

# Aggregations
df['total_purchase'] = df['quantity'] * df['unit_price']

# Log transform (for skewed data)
df['log_income'] = np.log1p(df['income'])
```

### Date/Time Features

```python
df['date'] = pd.to_datetime(df['date'])

# Extract components
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day_of_week'] = df['date'].dt.dayofweek
df['is_weekend'] = df['day_of_week'].isin([5, 6]).astype(int)

# Cyclical encoding
df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
```

### Text Features

```python
# Length features
df['text_length'] = df['text'].str.len()
df['word_count'] = df['text'].str.split().str.len()

# TF-IDF
from sklearn.feature_extraction.text import TfidfVectorizer
tfidf = TfidfVectorizer(max_features=100)
text_features = tfidf.fit_transform(df['text'])
```

## Feature Selection

### Removing Low Variance Features

```python
from sklearn.feature_selection import VarianceThreshold

selector = VarianceThreshold(threshold=0.01)
X_selected = selector.fit_transform(X)
```

### Correlation Analysis

```python
# Remove highly correlated features
correlation_matrix = df.corr().abs()
upper = correlation_matrix.where(
    np.triu(np.ones(correlation_matrix.shape), k=1).astype(bool)
)
to_drop = [col for col in upper.columns if any(upper[col] > 0.95)]
```

### Feature Importance

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier()
rf.fit(X, y)

importance = pd.DataFrame({
    'feature': X.columns,
    'importance': rf.feature_importances_
}).sort_values('importance', ascending=False)
```

## Best Practices

1. **Understand your data** before engineering
2. **Create features iteratively** - start simple
3. **Validate with cross-validation** - avoid leakage
4. **Document everything** - feature definitions matter
5. **Automate pipelines** - use sklearn Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

preprocessor = ColumnTransformer([
    ('num', StandardScaler(), numeric_features),
    ('cat', OneHotEncoder(), categorical_features)
])

pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier())
])
```

## Conclusion

Good feature engineering requires both technical skills and domain expertise. Invest time in understanding your data, and your models will reward you with better performance.
