
## Recipie Recommendation System - Content - Based 
### Exploratory analysis and prediction
### Alicja Wilk
#### Datalytics with Alicja 

# Data Preparation and Exploration
Began with reading in the 3 files provided and exploring the data.
    - core-data_recipe read into dfRecipe
    - core-data-train_rating read into dfTrain
    - core-data-test_rating read into dfTest


```python
# checking cd
import os 
os.chdir ("/Users/alicjawilk/Desktop/datascience/projects/ml_projects/data/data")
print(os.getcwd())
#print(os.listdir(os.getcwd()))
```

    /Users/alicjawilk/Desktop/datascience/projects/ml_projects/data/data



```python
import pandas as pd
import re

import matplotlib.pyplot as plt
%matplotlib inline

```


```python
n_row = None
df = pd.read_csv('core-data_recipe.csv',nrows = n_row)
nRow, nCol = df.shape
print(f"There are {nRow} rows, and {nCol} columns in the core data recipie file.")

n_row = None
df_test = pd.read_csv('core-data-test_rating.csv',nrows = n_row)
nRow, nCol = df_test.shape
print(f"There are {nRow} rows, and {nCol} columns in the test rating recipie file.")

n_row = None
df_train = pd.read_csv('core-data-train_rating.csv',nrows = n_row)
nRow, nCol = df_test.shape
print(f"There are {nRow} rows, and {nCol} columns in the train rating recipie file.")

```

    There are 45630 rows, and 6 columns in the core data recipie file.
    There are 283440 rows, and 4 columns in the test rating recipie file.
    There are 283440 rows, and 4 columns in the train rating recipie file.



```python
df.head(2)
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
      <th>recipe_id</th>
      <th>recipe_name</th>
      <th>image_url</th>
      <th>ingredients</th>
      <th>cooking_directions</th>
      <th>nutritions</th>
      <th>avg_rating</th>
      <th>count</th>
      <th>weighted_rating</th>
      <th>ingredients_new</th>
      <th>ingredientsTokenized</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>240488</td>
      <td>Pork Loin, Apples, and Sauerkraut</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>sauerkraut drained^Granny Smith apples sliced^...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n2 h 30 m\nR...</td>
      <td>{u'niacin': {u'hasCompleteData': False, u'name...</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>[sauerkraut drained, granny smith apples slice...</td>
      <td>[sauerkraut, drained, granny, smith, apples, s...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>218939</td>
      <td>Foolproof Rosemary Chicken Wings</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>chicken wings^sprigs rosemary^head garlic^oliv...</td>
      <td>{'directions': u"Prep\n20 m\nCook\n40 m\nReady...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>4.291218</td>
      <td>[chicken wings, sprigs rosemary, head garlic, ...</td>
      <td>[chicken, wings, sprigs, rosemary, head, garli...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_test.head(2)
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
      <th>user_id</th>
      <th>recipe_id</th>
      <th>rating</th>
      <th>dateLastModified</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5215572</td>
      <td>55090</td>
      <td>5</td>
      <td>2015-01-09T18:05:22.95\n</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5215572</td>
      <td>26317</td>
      <td>4</td>
      <td>2016-12-04T17:50:35.777\n</td>
    </tr>
  </tbody>
</table>
</div>




```python
train_mean = df_train.groupby('recipe_id').rating.mean()
train_count = df_train.groupby('recipe_id').rating.count()
print(train_mean.head())
print(train_count.head())
print('Average # of recipes rated by users in training: ', train_mean.mean())
```

    recipe_id
    6663    4.315789
    6664    4.375000
    6665    3.666667
    6666    4.696429
    6667    4.333333
    Name: rating, dtype: float64
    recipe_id
    6663    19
    6664    16
    6665    18
    6666    56
    6667    24
    Name: rating, dtype: int64
    Average # of recipes rated by users in training:  4.269739317278455



```python
#Plotting # of receipes rated by users in a histogram
train_count.plot.hist(grid=True, bins=30, rwidth=0.9,
                   color='#607c8e')
plt.title('# of Recipes Rated by each User')
plt.xlabel('Users')
plt.ylabel('Recipe Counts')
plt.ylim([0, 1500])
plt.grid(axis='y', alpha=0.75)
```


![png](output_8_0.png)



```python
train_mean.plot.hist(grid=True, bins=30, rwidth=0.9,
                   color='#607c8e')
plt.title('# of Recipes Rated by each User')
plt.xlabel('Users')
plt.ylabel('Recipe Counts')
plt.ylim([0, 1500])
plt.grid(axis='y', alpha=0.75)
```


![png](output_9_0.png)


#### Adding Features to Recipes
* Average rating per recipe
* Count of # of ratings for each recipe


```python
df = pd.merge(df, train_mean, how = 'outer', on = 'recipe_id')
df = pd.merge(df, train_count, how = 'outer', on = 'recipe_id')
df = df.fillna(train_mean.mean())
df.rename(columns={'rating_x':'avg_rating',
                          'rating_y':'count',
                          },inplace=True)
print(df.shape)
df.head(3)
```

    (45630, 8)





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
      <th>recipe_id</th>
      <th>recipe_name</th>
      <th>image_url</th>
      <th>ingredients</th>
      <th>cooking_directions</th>
      <th>nutritions</th>
      <th>avg_rating</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>240488</td>
      <td>Pork Loin, Apples, and Sauerkraut</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>sauerkraut drained^Granny Smith apples sliced^...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n2 h 30 m\nR...</td>
      <td>{u'niacin': {u'hasCompleteData': False, u'name...</td>
      <td>4.269739</td>
      <td>4.269739</td>
    </tr>
    <tr>
      <th>1</th>
      <td>218939</td>
      <td>Foolproof Rosemary Chicken Wings</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>chicken wings^sprigs rosemary^head garlic^oliv...</td>
      <td>{'directions': u"Prep\n20 m\nCook\n40 m\nReady...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>5.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>87211</td>
      <td>Chicken Pesto Paninis</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>focaccia bread quartered^prepared basil pesto^...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n5 m\nReady ...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>4.617647</td>
      <td>34.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### Weighted Rating per  Recipe
* Adding a weighted rating for the average recipes which factors in the number of ratings that were used to determine that rating
* This increases average ratings for receips with higher counts and lowers average ratings for those recipes with few counts.


```python
recipec = df['avg_rating'].mean()
recipem = df['count'].quantile(.9)
print(recipec)
print(recipem)
```


```python
def weighted_rating(x, m=recipem, c=recipec):
    v = x['count']
    R = x['avg_rating']
    
    return (v/(v+m) * R) + (m/(m+v) * c)
```


```python
df['weighted_rating'] = df.apply(weighted_rating, axis = 1)
df.head(2)
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
      <th>recipe_id</th>
      <th>recipe_name</th>
      <th>image_url</th>
      <th>ingredients</th>
      <th>cooking_directions</th>
      <th>nutritions</th>
      <th>avg_rating</th>
      <th>count</th>
      <th>weighted_rating</th>
      <th>ingredients_new</th>
      <th>ingredientsTokenized</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>240488</td>
      <td>Pork Loin, Apples, and Sauerkraut</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>sauerkraut drained^Granny Smith apples sliced^...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n2 h 30 m\nR...</td>
      <td>{u'niacin': {u'hasCompleteData': False, u'name...</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>[sauerkraut drained, granny smith apples slice...</td>
      <td>[sauerkraut, drained, granny, smith, apples, s...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>218939</td>
      <td>Foolproof Rosemary Chicken Wings</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>chicken wings^sprigs rosemary^head garlic^oliv...</td>
      <td>{'directions': u"Prep\n20 m\nCook\n40 m\nReady...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>4.291218</td>
      <td>[chicken wings, sprigs rosemary, head garlic, ...</td>
      <td>[chicken, wings, sprigs, rosemary, head, garli...</td>
    </tr>
  </tbody>
</table>
</div>



# Feature Engineering (continued)
II.  Ingredients

* Begin by identifying key singular ingredients, in the ingredients column, that are two words and combine the words with a hyphen. For example, "olive oil" will be converted to "olive-oil." 


```python
def hyphenate_ingredients(df):
    df['ingredients'].replace(regex=r'baking soda',value='baking-soda',inplace=True)
    df['ingredients'].replace(regex=r'baking powder',value='baking-powder',inplace=True)
    df['ingredients'].replace(regex=r'sesame seeds',value='sesame-seeds',inplace=True)
    df['ingredients'].replace(regex=r'simple syrup',value='simple-syrup',inplace=True)
    df['ingredients'].replace(regex=r'olive oil',value='olive-oil',inplace=True)
    df['ingredients'].replace(regex=r'corn starch',value='corn-starch',inplace=True)
    df['ingredients'].replace(regex=r'garam masala',value='garam-masala',inplace=True)
    df['ingredients'].replace(regex=r'balsamic vinegar',value='balsamic-vinegar',inplace=True)
    df['ingredients'].replace(regex=r'sour cream',value='sour-cream',inplace=True)
    df['ingredients'].replace(regex=r'red bell pepper',value='red bell-pepper',inplace=True)
    df['ingredients'].replace(regex=r'green bell pepper',value='green bell-pepper',inplace=True)
    df['ingredients'].replace(regex=r'garam masala',value='garam-masala',inplace=True)
    df['ingredients'].replace(regex=r'peanut butter',value='peanut-butter',inplace=True)
    df['ingredients'].replace(regex=r'cream cheese',value='cream-cheese',inplace=True)
    df['ingredients'].replace(regex=r'garlic powder',value='garlic-powder',inplace=True)
    
  
    return df
```


```python
df = hyphenate_ingredients(df)
df.head(2)
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
      <th>recipe_id</th>
      <th>recipe_name</th>
      <th>image_url</th>
      <th>ingredients</th>
      <th>cooking_directions</th>
      <th>nutritions</th>
      <th>avg_rating</th>
      <th>count</th>
      <th>weighted_rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>240488</td>
      <td>Pork Loin, Apples, and Sauerkraut</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>sauerkraut drained^Granny Smith apples sliced^...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n2 h 30 m\nR...</td>
      <td>{u'niacin': {u'hasCompleteData': False, u'name...</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>4.269739</td>
    </tr>
    <tr>
      <th>1</th>
      <td>218939</td>
      <td>Foolproof Rosemary Chicken Wings</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>chicken wings^sprigs rosemary^head garlic^oliv...</td>
      <td>{'directions': u"Prep\n20 m\nCook\n40 m\nReady...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>4.291218</td>
    </tr>
  </tbody>
</table>
</div>



#### Cleaning the ingredients list

Now we will clean the test in the "ingredients" column. First we create a new column called 'new_ingredients' to update and place the cleaned ingredients text. To clean the ingredients text, we will create a function called "cleanIngredients" with the Recipe dataframe as the argument.

Steps to clean ingredients text (using regex):
* Take out all uneccessary punctuation or special characters and replace with a space
* Remove all parenthetical phrases. E.g. "(ounces)" 
* Remove all "&"
* Remove all words that preceed a ":". These words tend to be a header for sub-ingredients. E.g. "Sauce: heavy whipping cream, butter, minced garlic, grated Parmesan cheese"
* Convert all text to lowercase characters


```python
df['ingredients_new'] = df['ingredients']
```


```python
def cleanIngredients(df):
    df['ingredients_new'] = df['ingredients_new'].str.replace(r"[^A-Za-z0-9,\(\):!?@\'\`\"\_\n\^\-]", " ")
    df['ingredients_new'] = df['ingredients_new'].str.replace(r"(\(([\w\s-]*)\))", "")
    df['ingredients_new'] = df['ingredients_new'].str.replace(r"&","")
    df['ingredients_new'] = df['ingredients_new'].str.replace(r"\^[A-Za-z\s]+:|^[A-Za-z\s]+:\^","")
    df['ingredients_new'] = df['ingredients_new'].str.replace(r"[0-9]\s*[0-9]+ | [0-9]+\-"," ")
    df['ingredients_new'] = df['ingredients_new'].str.lower()
    
    return df
```


```python
# Use the function to remove the extraneous items in the ingredients list
df = cleanIngredients(df)
```

#### Cleaning the ingredients list (continued)
* Remove the " ^ " character that is splitting the words
* Tokenize the cleaned-up ingredients for preprocessing for vectorization later on


```python
# Split the ingredients using the '^' delimiter that existed
df['ingredients_new'] = df['ingredients_new'].str.split("^", )
df.head()

# function to tokenize words within list pased in
def tokenizeList(l):
    new_l = [val.split(" ") for val in l]
    flat_l = [item for sublist in new_l for item in sublist]
    return flat_l

#Apply tokenize function to cleaned ingredients
df['ingredientsTokenized'] = df['ingredients_new'].apply(tokenizeList)
```

# Feature Engineering (continued)

III.  Adding lengths of ingredients and of directions to receipe dataframe


```python
# Creating a single string from the list of cleaned ingredients
df['ingredients'] = df['ingredients_new'].str.join(' ')
print(df['ingredients_new'].head())
print("\n")
print(df['ingredients'].head())

df['lengthDirections'] = df['cooking_directions'].apply(len)
df['lengthIngredients'] = df['ingredients'].apply(len)
df.head(2)
```

    0    [sauerkraut drained, granny smith apples slice...
    1    [chicken wings, sprigs rosemary, head garlic, ...
    2    [focaccia bread quartered, prepared basil pest...
    3    [red potatoes, strips bacon, heavy whipping cr...
    4    [skinless boneless chicken breast halves, dice...
    Name: ingredients_new, dtype: object
    
    
    0    sauerkraut drained granny smith apples sliced ...
    1    chicken wings sprigs rosemary head garlic oliv...
    2    focaccia bread quartered prepared basil pesto ...
    3    red potatoes strips bacon heavy whipping cream...
    4    skinless boneless chicken breast halves diced ...
    Name: ingredients, dtype: object





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
      <th>recipe_id</th>
      <th>recipe_name</th>
      <th>image_url</th>
      <th>ingredients</th>
      <th>cooking_directions</th>
      <th>nutritions</th>
      <th>avg_rating</th>
      <th>count</th>
      <th>weighted_rating</th>
      <th>ingredients_new</th>
      <th>ingredientsTokenized</th>
      <th>lengthDirections</th>
      <th>lengthIngredients</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>240488</td>
      <td>Pork Loin, Apples, and Sauerkraut</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>sauerkraut drained granny smith apples sliced ...</td>
      <td>{'directions': u'Prep\n15 m\nCook\n2 h 30 m\nR...</td>
      <td>{u'niacin': {u'hasCompleteData': False, u'name...</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>4.269739</td>
      <td>[sauerkraut drained, granny smith apples slice...</td>
      <td>[sauerkraut, drained, granny, smith, apples, s...</td>
      <td>833</td>
      <td>182</td>
    </tr>
    <tr>
      <th>1</th>
      <td>218939</td>
      <td>Foolproof Rosemary Chicken Wings</td>
      <td>https://images.media-allrecipes.com/userphotos...</td>
      <td>chicken wings sprigs rosemary head garlic oliv...</td>
      <td>{'directions': u"Prep\n20 m\nCook\n40 m\nReady...</td>
      <td>{u'niacin': {u'hasCompleteData': True, u'name'...</td>
      <td>5.000000</td>
      <td>1.000000</td>
      <td>4.291218</td>
      <td>[chicken wings, sprigs rosemary, head garlic, ...</td>
      <td>[chicken, wings, sprigs, rosemary, head, garli...</td>
      <td>936</td>
      <td>78</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Conversion function used to convert hours:minutes to total minutes
# and had to account for cases where one, the other or both exist
def TimeConvert(time1,time2):
    HorM = ''
    if time2 != 0:
        HorM = re.search(r'[h,m]',time2).group()
        time2 = re.search(r'\d+', time2).group()
        if time1 != 0:
            time1 = re.search(r'\d+', time1).group()
            return ((int(time1))*60+int(time2))
        else:
            if HorM == 'm':
                return (time2)
            else:
                return (int(time2)*60)
    else: 
        return(0)
```


```python

```
