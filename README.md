mletrics
================

<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

## Install

`pip install mletrics`

## How to use

### Calculating psi values

``` python
from mletrics.stability import psi
```

``` python
import numpy as np
import pandas as pd
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OneHotEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
```

``` python
from pathlib import Path

p = Path('..')
df = pd.read_csv(p/'datasets/titanic.csv')
df.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>PassengerId</th>
      <th>Survived</th>
      <th>Pclass</th>
      <th>Name</th>
      <th>Sex</th>
      <th>Age</th>
      <th>SibSp</th>
      <th>Parch</th>
      <th>Ticket</th>
      <th>Fare</th>
      <th>Cabin</th>
      <th>Embarked</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>Braund, Mr. Owen Harris</td>
      <td>male</td>
      <td>22.0</td>
      <td>1</td>
      <td>0</td>
      <td>A/5 21171</td>
      <td>7.2500</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>Cumings, Mrs. John Bradley (Florence Briggs Th...</td>
      <td>female</td>
      <td>38.0</td>
      <td>1</td>
      <td>0</td>
      <td>PC 17599</td>
      <td>71.2833</td>
      <td>C85</td>
      <td>C</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1</td>
      <td>3</td>
      <td>Heikkinen, Miss. Laina</td>
      <td>female</td>
      <td>26.0</td>
      <td>0</td>
      <td>0</td>
      <td>STON/O2. 3101282</td>
      <td>7.9250</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>Futrelle, Mrs. Jacques Heath (Lily May Peel)</td>
      <td>female</td>
      <td>35.0</td>
      <td>1</td>
      <td>0</td>
      <td>113803</td>
      <td>53.1000</td>
      <td>C123</td>
      <td>S</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>0</td>
      <td>3</td>
      <td>Allen, Mr. William Henry</td>
      <td>male</td>
      <td>35.0</td>
      <td>0</td>
      <td>0</td>
      <td>373450</td>
      <td>8.0500</td>
      <td>NaN</td>
      <td>S</td>
    </tr>
  </tbody>
</table>
</div>

``` python
cat_vars = ['Pclass', 'Sex', 'Embarked']
num_vars = ['Age', 'SibSp', 'Fare']
features = cat_vars + num_vars
target = 'Survived'

X = df[features].copy()
y = df[target].copy()
```

``` python
num_pipe = Pipeline(steps=[
        ('imputer', SimpleImputer(strategy='constant', fill_value=-999))
])

cat_pipe = Pipeline(steps=[
        ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
        ('ohe', OneHotEncoder(sparse=False, handle_unknown='ignore'))
]) 

transformers = ColumnTransformer(transformers=[
                ('numeric', num_pipe, num_vars),
                ('categoric', cat_pipe, cat_vars)
])

model = Pipeline(steps=[
        ('transformers', transformers),
        ('model', RandomForestClassifier(random_state=42, max_depth=3))
])
```

``` python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

``` python
model.fit(X_train, y_train)

y_proba_train = model.predict_proba(X_train)[:,1]
y_proba_test  = model.predict_proba(X_test)[:,1]
```

``` python
# calculate psi value for the model probability between train and test
psi(y_proba_train, y_proba_test)
```

    0.06001324825109782

-   PSI \< 0.1 - No change. You can continue using existing model.
-   PSI \>= 0.1 but less than 0.2 - Slight change is required.
-   PSI \>= 0.2 - Significant change is required. Ideally, you should
    not use this model any more.

Reference:
https://www.listendata.com/2015/05/population-stability-index.html
