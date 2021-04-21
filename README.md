
# Big Data Final Project - Chase Staples
## Word Count - Text Processing

## Source


## Languages, Libraries and Tools

Languages : Python

Tools: Pyspark, Databricks Notebook

Modules/Libraries: nltk, regex, seaborn, matplotlib, pandas

1. Import libraries/modules and tools

```Python
# pyspark to remove stopwords (eg: the, and, etc..)
from pyspark.ml.feature import StopWordsRemover
# regex to remove puncuations
import re
# plot the graphs with matplotlib, seaborn and panda
import matplotlib as plt
import pandas as pd
import seaborn as sns
```

2. Getting our "data" or text to be processes

```Python
# grabbing the raw text from elsewhere
import urllib.request
urllib.request.urlretrieve("https://raw.githubusercontent.com/s528345/staples-big-data-final-project/main/TheGreatGatsby.txt", "/tmp/Gatsby.txt")
```
3. Now the data is stored, we will move the data into a temporary file into a different location using dbutils.fs.mv. dbutils.fs.mv works by taking two arguements. The first is the name of file we want to move, the second is where we want the file to be moved too or where the file will be stored as. In this particular case we will move the file from "tmp" to "data" as shown.

```Python
dbutils.fs.mv("file:/tmp/Gatsby.txt","dbfs:/data/Gatsby.txt")
```

4. Finally, we will transfer this data file into Spark. To do this, take the data file we have now and converting it into Spark's RDD  or Resilient Distributed Datasets. As shown using sc.textFile.

```Python
gatsbyRDD = sc.textFile("dbfs:/data/Gatsby.txt")
```

## "Cleaning" the Data

We now have the data we want to clean or take away punctuations, and stopwords (words that don't add much to or convey little meaning to a sentence eg: the, and, etc.).

5. To "Clean" the data we will change the words to either lowercase or uppercase, remove spaces and then split the words

```Python
# Puting words into a list but putting them to lowercase and then split by spaces
wordsRDD = gatsbyRDD.flatMap(lambda x: x.upper().strip().split(" "))
# Using a lambda function to "clean" the words for better processing
cleanTokensRDD = wordsRDD.map(lambda x: re.sub(r'[^a-zA-Z]','',x))
```

6. Next is removing stopwords from StopWordsRemover from pyspark.ml.feature.

```Python
remover = StopWordsRemover()
stopwords = remover.getStopWords()
cleanWordsRDD = cleanTokensRDD.filter(lambda x: x not in stopwords)
```

## Processing the data further

7. Map the words into key:value pairs 

```Python
IKVPairsRDD= cleanWordsRDD.map(lambda word: ( word , 1 ))
```

8. Then, we use a word count to count the number of words as it is being used

```Python
wordCountRDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc + value)
```

9. Finally taking that list of words and putting it into reverse/descending order using sortByKey and then taking the top 25

```Python
results = wordCountRDD.map(lambda x: (x[1], x[0])).sortByKey(False).take(25)
```

## Graphing the data

10 To be able to see the data we will use MatPlotLib, Seadborn, and Pandas

```Python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

source = 'The Great Gatsby'
title = 'Top Words in ' + source
xlabel = 'Count'
ylabel = 'Words'

df = pd.DataFrame.from_records(results, columns =[xlabel, ylabel]) 
plt.figure(figsize=(25,10))
sns.barplot(xlabel, ylabel, data=df, palette="Paired").set_title(title)
```

![](Graph.png)













