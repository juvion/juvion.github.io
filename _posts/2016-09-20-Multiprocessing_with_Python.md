---
layout: post
title:  "Multiprocessing with Python."
date:   2016-09-20
---


### Parallel computing using Python  
This is a simple tutorial of how to utilize Python's `multiprocessing` library to perform parallel tasks (using multiple CPUs). 


```python
#import the libraries
import pandas as pd
import multiprocessing
```

The example dataset is [`samplesTests.csv`](https://github.com/juvion/juvion.github.io/raw/master/ref/samplesTests.zip), which contains the testing timestamps and sample IDs, and it has 886413 tests in total.


```python
dfTest = pd.read_csv("./samplesTests.csv")
```


```python
dfTest.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TestTime</th>
      <th>SampleID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8/10/2004 1:15</td>
      <td>S00802</td>
    </tr>
    <tr>
      <th>1</th>
      <td>8/10/2004 1:15</td>
      <td>S00802</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8/10/2004 1:16</td>
      <td>S00802</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8/10/2004 1:24</td>
      <td>S00802</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8/10/2004 1:24</td>
      <td>S00802</td>
    </tr>
    <tr>
      <th>5</th>
      <td>8/10/2004 0:46</td>
      <td>S96030</td>
    </tr>
    <tr>
      <th>6</th>
      <td>8/10/2004 0:46</td>
      <td>S96030</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8/10/2004 0:46</td>
      <td>S96030</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8/10/2004 0:46</td>
      <td>S96030</td>
    </tr>
    <tr>
      <th>9</th>
      <td>8/10/2004 4:41</td>
      <td>S00619</td>
    </tr>
  </tbody>
</table>
</div>




```python
dfTest.shape
```




    (886413, 2)



What I want to achieve here is to count how many tests have been performed for each sample, and I like to distribute this work into multiple tasks based on sample IDs. 
So the first thing is to get the list of sample IDs.


```python
samples = dfTest['SampleID'].unique()
samples
```




    array(['S00802', 'S96030', 'S00619', ..., 'S00267', 'S00014', 'S01194'], dtype=object)



Then define a function `test_counter()`, which is the job that each thread of the parallel tasks will be performing. It counts the size of dataframe that grouped by the sampleID, and stores that count into a dictionary.


```python
#initiate a test_counts dictionary
test_counts = {}
def test_counter(sample):
    #Subset a dataframe for selected sampleID
    dfTest_ = dfTest[dfTest['sampleID'] == sample]
    #get the rows of the selected subset of the dataframe
    test_count = dfTest_.shape[0]
    #add the count into the test_counts dictionary
    test_counts[sample] = test_count
    return test_counts
```

Now, the parallel computing needs another function, which serves as multiple task handler. It assigns the number of threads, and arranges the parallel tasks. 
`map` function takes two parameters:  
- a function: the task   
- a list: list of inputs for the task function.   
For example:


```python
#initiate an instance of Pool
p = multiprocessing.Pool()
#run the parallel tasks
result_L = p.map(test_counter, samples)
```

There is a tiny side effect of this particular parallel computing. `result_L` ends as a list of dictionaries, which makes sense... `Pool` will collect all parallel tasks' results (dictionaries) into a list. So, I have to merge the list of dictionaries into a large dictionary.


```python
#merge all the dictionaries in the list into a large dictionary
for d in result_L:
    results.update(d)
```

So put all together, following is the full code. 
`python test_counts.py`

```python
import multiprocessing
import pandas as pd

# read in the data
dfTest = pd.read_csv("./samplesTests.csv")
test_counts = {}

#get the list of all sampleIDs
samples = dfTest['SampleID'].unique()

#initiate a test_counts dictionary
test_counts = {}
def test_counter(sample):
    #Subset a dataframe for selected sampleID
    dfTest_ = dfTest[dfTest['sampleID'] == sample]
    #get the rows of the selected subset of the dataframe
    test_count = dfTest_.shape[0]
    #add the count into the test_counts dictionary
    test_counts[sample] = test_count
    return test_counts

def mp_handler():
    results = {}
    #initiate an instance of Pool
    p = multiprocessing.Pool()
    #run the parallel tasks
    result_L = p.map(test_counter, samples)
    
    #merge all the dictionaries in the list into a large dictionary
    for d in result_L:
        results.update(d)

    for k, v in results.iteritems():
        print k, v

if __name__ == '__main__':
    mp_handler()
```

Note Functionality within the package of `multiprocessing` requires that the `\_\_main\_\_` module be importable by the children [1](https://docs.python.org/2/library/multiprocessing.html#multiprocessing-programming). This means that some examples, such as the Pool examples do not work in the interactive interpreter (so does jupyter notebook). For example:

```
>>> from multiprocessing import Pool
>>> p = Pool(5)
>>> def f(x):
...     return x*x
...
>>> p.map(f, [1,2,3])
Process PoolWorker-1:
Process PoolWorker-2:
Process PoolWorker-3:
Traceback (most recent call last):
AttributeError: 'module' object has no attribute 'f'
AttributeError: 'module' object has no attribute 'f'
AttributeError: 'module' object has no attribute 'f'
```

For further reading, you can go to here [2](https://pymotw.com/2/multiprocessing/communication.html)

### reference
1. [multiprocessing documentation](https://docs.python.org/2/library/multiprocessing.html#multiprocessing-programming)    
2. [Communication Between Processes](https://pymotw.com/2/multiprocessing/communication.html)  
