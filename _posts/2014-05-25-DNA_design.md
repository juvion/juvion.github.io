---
layout: post
title:  "Python script: Automate DNA sequences design."
date:   2014-05-25
---

### Background  
Beyond its central role in biology, DNA was also known to provide switchable nanomechanical devices
whose equilibrium states can be controlled by adding ligands or complementary DNA fragment.
This feature makes DNA switch design the core of many promising nanoscale applications,such as probes
for detecting pathogens or nanosensor for chemical ligands.


In this project, two types of DNA switches have been designed,where the switches will change the
conformation with adding analyte DNA or ATP. The product structure of the switches will expose a
single strand available for hybridization to a molecular beacon, for which hybridization can be
detected by fluorescence. With the scheme’s structure information, the design program from our
lab was used to search for candidate sequences to fit the structural constraints with low ensemble
defect. To ensure that the DNA switch transits to the ligand-binding product efficiently, free
energy prediction is performed to evaluate the equilibrium constants. Two sets of DNA switches
with the corresponding molecular beacons sequences have been designed. The analyte DNA switch has
been recently positively confirmed, and the ATP switch’s test is under going.

![DNA_switch_scheme][1]

#### python script to patch the sequences based on thermal energy optimization.  

```python
#!/usr/bin/python
__author__ = 'ju'
"""
This python script is applied to generate all species sequences with
determined part for the ATP aptamer switch, based on the input: Aptamer.
"""

import string
import sys
import subprocess

#presettings
aptamer = 'CTGGGGGAGTATTGCGGAGGAAG'
nano_switch_size = 56
beacon_size = 16


#function to create complement sequence.
def complement(input_seq):
    complement_seq = input_seq.translate(string.maketrans('UTAGCutagc', 'AATCGAATCG'))[::-1]
    return complement_seq

#function to link the multiple sequences by 'AAA' to create a new sequence for evaluation.
#*args is a way to take variable parameters.
def link_seqs(*input_seqs):
    output_seq = 'AAA'.join(input_seqs)
    print output_seq

#function to push input sequence into a input ct file, e.g, change the sequence of ct file as input seq
def push_seq2ct(input_seq, input_ct_file, output_ct_file):
    fo = open(output_ct_file, 'w')
    with open(input_ct_file, 'r') as fi:
        #read header of input ct and write it into output ct.
        fo.writelines(next(fi))
        i = 0
        for line in fi:
            entries = line.split()
            fo.writelines(entries[0] + '\t' + input_seq[i] + '\t' +'\t'.join(entries[2:]) + '\n')
            i += 1


#function to get seq string from a input ct file
def get_ct_seq(input_ct_file):
    with open(input_ct_file, 'r') as fi:
        #read header of input ct and write it into output ct.
        next(fi)
        seq = ''
        i = 0
        for line in fi:
            entries = line.split()
            seq += entries[1]
            i += 1
    return(seq)

#function for two aptamers to determine the beacon proto sequence.
def beacon_patch(aptamer1_bind_seq, aptamer2_bind_seq):
    #create the proto sequence of nano_switch and beacon, filled with 'N'.
    #The sequence is assigned as bytearray, which is easier for partial assignment
    proto_nano_switch = bytearray('N' * nano_switch_size)
    # proto_beacon = bytearray('N' * beacon_size)
    #patch the proto_nano_switch from starting to the ending of bases, by sequence complementary feature.
    #State1: DA1 and DA2 binding part
    proto_nano_switch[4:10] = complement(aptamer1_bind_seq)
    proto_nano_switch[10:16] = aptamer2_bind_seq
    #State 2: Hairpin part, proto_nano_switch[8:19] now is changed from starting sequence
    proto_nano_switch[25:36] = complement(proto_nano_switch[8:19])
    #State 1: Hairpin after bulge
    proto_nano_switch[41:47] = complement(proto_nano_switch[31:37])
    #State 2: Hairpin before bulge
    proto_nano_switch[47:55] = complement(proto_nano_switch[22:30])
    #predisign beacon is the sequence for beacon designing input
    predesign_beacon = complement(proto_nano_switch[40:56])

    #return proto_nano_switch and predesign_beacon as string.
    return (str(proto_nano_switch), str(predesign_beacon))

#function to patch the beacon to proto_nano_switch, which will be used for nano_switch's design.
def nano_switch_patch(designed_beacon, proto_nano_switch):
    #The sequence is assigned as bytearray, which is easier for partial assignment
    predesign_nano_switch = bytearray(proto_nano_switch)
    #first patch the designed beacon sequence to predesign_nano_switch tail.
    predesign_nano_switch[40:56] = complement(designed_beacon)
    #State 2: Hairpin before bulge
    predesign_nano_switch[22:30] = complement(predesign_nano_switch[47:55])
    #State 1: Hairpin after bulge
    predesign_nano_switch[31:37] = complement(predesign_nano_switch[41:47])
    #State 2: Hairpin part, proto_nano_switch[8:19] now is changed from starting sequence
    predesign_nano_switch[8:19] = complement(predesign_nano_switch[25:36])
    return str(predesign_nano_switch)

#call shell to run design program.
def design(input_ct_file, output_file_prefix, output_seq_num):
    for i in range(int(output_seq_num)):
        subprocess.call(['design', input_ct_file, '1', '2', '2', '0.03', '50',
                        output_file_prefix + str(i) + '.ct', '10', '4', '20', '1'])

# def evaluation():
#     # cmd = subprocess.Popen('gls -v', shell=True, stdout=subprocess.PIPE)
#     # for line in cmd.stdout:
#     #     if "ct" in line:
#     #         print line

###########
#creat list for proto nano switches and beacons.
proto_ns_list = []
proto_bc_list = []
#loop over the two possible 6 nt length aptamer binding sequences, and append the output
#nano switches and beacons into their lists.
for i in range(len(aptamer) - 5):
    for j in range(len(aptamer) - 5):
        aptamer1_bind_seq = aptamer[i:i + 6]
        aptamer2_bind_seq = aptamer[j:j + 6]
        # fo_ns_name = 'proto_ns_' + i + '_' + j
        proto_ns_list.append(beacon_patch(aptamer1_bind_seq, aptamer2_bind_seq)[0])
        proto_bc_list.append(beacon_patch(aptamer1_bind_seq, aptamer2_bind_seq)[1])

#########
# loop over all proto_beacons sequences and enforce the sequence to beacon_temp ct file.
# generate proto_bc_ct file.
m = 0
for proto_bc in proto_bc_list:
    # enforce the proto_bc sequence into the proto_bc's ct file.
    proto_bc_ct_file = './beacon/proto_beacon_' + str(m) + '.ct'
    push_seq2ct(proto_bc, sys.argv[1], proto_bc_ct_file)
    m += 1

##########
subprocess.call('export DATAPATH=/Users/ju/Sources/RNAstructureZero/data_tables', shell=True)
# run design for beacon.
for i in range(len(proto_bc_list)):
    proto_bc_ct_file = './beacon/proto_beacon_' + str(i) + '.ct'
    designed_bc_ct_prefix = './beacon/designed_beacon_proto_' + str(i) + '_run'
    design(proto_bc_ct_file, designed_bc_ct_prefix, 5)

```


[1]: https://dl.dropboxusercontent.com/u/3637996/github_pages/post_2013-05-25-DNAdesign/DNAdesign.png
