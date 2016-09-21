---
layout: post
title:  "Multiprocessing with Python."
date:   2016-09-20
---


So put all together, following is the full code. Note that iPython is not able to call multiple CPUs, and it has to run in terminal. Save the code as `test_counts.py` in your working directory, and put `samplesTests.csv` in the same folder with the code, then run
`python test_counts.py`

```python
import multiprocessing
import pandas as pd

# read in the data
dfTest = pd.read_csv("./samplesTests.csv")
test_counts = {}

samples = dfTest['sampleID'].unique()


def test_counter(sample):
    dfTest_ = dfTest[dfTest['sampleID'] == sample]
    test_count = dfTest_.shape[0]
    test_counts[sample] = test_count
    return test_counts


def mp_handler():
    results = {}
    p = multiprocessing.Pool()
    result_L = p.map(test_counter, samples)
    for d in result_L:
        results.update(d)
    for k, v in results.iteritems():
        print k, v


if __name__ == '__main__':
    mp_handler()
```