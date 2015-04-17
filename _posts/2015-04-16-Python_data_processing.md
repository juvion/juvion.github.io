---
layout: post
title:  "Processing multi-dimension data with Python."
date:   2015-04-16
---

In this yeast tRNA project, we are interested to determine the mutations that are responsible for yeast temperature sensitiveness and RTD. We've grew four different biological conditions of yeast strands (28 degree vs 37 degree, met22- / met 22+), and each strand were repeated 2-4 times for flow cytometery and RNA-seq. GFP-seq score are measured based on cell counts and RNA-seq read counts to indicate the tRNA activity. The first issue to address is, what should we do with the replica? Taking the mean? How many data we will get, since not all the mutations have all the values. 

This python script takes batch index as input, and calculate the total counts of the variants that have the GFP-seq value in the combination.

pandas is a python module that is robust for data processing.  

```python

#!/opt/local/bin/python
'''
I got Ambry genetics intern position today. It's exciting! 
People who talked to me are really nice, 
and I am a little bit embarrassed that I failed to answer such simple questions ...
                                                                    -- Ju 2015.04.16
'''
import sys
import pandas as pd

fi = sys.argv[1]

#read in data locally
rdf = pd.read_csv(fi)

#filter to get only variants that has less than or equal to 2 mutants.
df = rdf[rdf['Mutation Count'] < 3]

batch_dict = {
             'batch1': 'delta1 reflow normalized GFPseq', 
             'batch2': 'delta2 reflow normalized GFPseq', 
             'batch3': 'WT1 reflow normalized GFPseq', 
             'batch4': 'WT2 reflow normalized GFPseq', 
             'batch5': '37deg reflow normalized GFPseq', 
             'batch6': '37deg 2 reflow normalized GFPseq', 
             'batch7': '37deg 3 reflow normalized GFPseq', 
             'batch8': '37deg 4 reflow normalized GFPseq', 
             'batch9': 'delta1 37deg reflow normalized GFPseq', 
             'batch10': 'delta2 37deg reflow normalized GFPseq'}

def cross_filter(df, batch1, batch2):
    variants_num = 0
    df1 = pd.notnull(df[batch_dict[batch1]])
    df2 = pd.notnull(df[batch_dict[batch2]])
    for i in range(len(df1)):
        if df1[i] and df2[i]:
            variants_num += 1
    return str(variants_num)


def cross_filter_m(df, batch_list):
    """cross check multiple batchs about the overlapped data entries.
    """

    variants_num = 0
    batchs = {}

    #create a batch dictionary, key is the index of batch, and value is the batch's name.
    for i in range(len(batch_list)):
        #the batch_data only collect the bool value of the entry is null or not.
        batch_data = pd.notnull(df[batch_dict[batch_list[i]]])
        batchs.update({('df' + str(i+1)): batch_data})


    #test the boolean value of the cross 
    for i in range(len(batchs['df1'])):
        #check is the bool value indicating the entry is null or not.
        check = True
        for key, value in batchs.iteritems():
            check = check and value[i]
            if not check:
                break

        if check:
            variants_num += 1
    return str(variants_num)



def main():
    index ="""
Batch indices:
1: delta1 
2: delta2 
3: WT1 
4: WT2 
5: 37deg 
6: 37deg 2 
7: 37deg 3 
8: 37deg 4 
9: delta1 37deg 
10: delta2 37deg """
    
    #print the matrix of 2 batches crossing counts.
    print "Map of numbers of overlapped vriants for all the possible two crossing batches\n"
    print '\t' + '\t'.join([str(i) for i in reversed(range(2, 11))])
    for i in range(1, 10):
        report = str(i) + '\t'
        for j in range(10, i, -1):
            batch1, batch2 = 'batch' + str(i), 'batch' + str(j)
            report += cross_filter(df, batch1, batch2) + '\t'
        print report + '\n'    

    while True:
        print index + '\n'
        batch_nums = []
        batch_nums = raw_input("Please choose the indexing numbers separated with space. ctrl + z to quit.\n").split()
        try:
            batch_list = ['batch' + batch_num for batch_num in batch_nums]
            print batch_list
            print cross_filter_m(df, batch_list) + " variants are overlapped. \n"
        except KeyError:
            print "Error! The number you chose is not in the range of 1-10. Please try again.\n...\n...\n"

main()

```