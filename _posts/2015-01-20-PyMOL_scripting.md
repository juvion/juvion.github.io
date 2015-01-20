---
layout: post
title:  "Scripting with PyMOL."
date:   2015-01-20
---

#Scripting with PyMOL

PyMOL supports Python scripting. This gives users the power to implement 
most of the Python libraries to write programs and then send the results 
back into PyMOL. Useful extensions to PyMOL can be found in our 
[Script Library][1]. 

I used PyMOL frequently when I was working on tRNA MD simulation. 
PyMOL is a very convenient tool to visualize PDB structure and do some 
molecular measurements.

In my current inhibitory codon pair (ICP) project, I need to examine a list of PDB 
structures to locate and highlight the ICP coded dipeptide. A relatively 
high throughput approach to use PyMOL will be necessary.

The detailed tutorial can be found from [PyMOLwiki][2]

>General PyMOL scripting is done in Python. It's really quite simple, 
just write your function (following a couple simple rules) and then 
let PyMOL know about it by using the cmd.extend command. 
Here's the simple recipe for writing your own simple scripts for PyMOL: 
> ####To write them:
>    1. Write the function, let's call it doSimpleThing, in a Python file, 
>    let's call the file pyProgram.py.
>    2. Add the following command to the end of the pyProgram.py file 
```python
cmd.extend("doSimpleThing",doSimpleThing)
```

> ####To use them:
>    1.  simply import the script into PyMOL console:  
```
run /home/userName/path/toscript/pyProgram.py
```
>    2.  Then, just type the name of the command: doSimpleThing and pass any needed arguments.  


mark_res.py  
```python
#2015 Xiaoju Zhang
from pymol import cmd


def mark_res():
    """ mark_res <pdb_file> <highlight_residue>
    select the chain and residue that ICP dipeptide locate and highlight it.
    """
    #list of pdb id, chain id, and residues informations for ICP.
    pdb_input_lists = [('1fa0', 'A', '177-178'),
                       ('1u5t', 'A', '65-66'),
                       ('1yke', 'A', '103-104'),
                       ('2b1e', 'A', '294-295'),
                       ('2ckz', 'D', '11-12'),
                       ('2pk9', 'D', '150-151'),                         
                       ('2pmi', 'D', '150-151'),
                       ('2xfv', 'A', '7-8'),
                       ('3esl', 'A', '191-192'),
                       ('3n7n', 'E', '20-21'),
                       ('3t5v', 'E', '288-289'),
                       ('4bh6', 'A', '365-366')]
    #iterate pdbs                   
    for pdb_info in pdb_input_lists:
        pdb_id = pdb_info[0]
        chain_id = pdb_info[1]
        res_id = pdb_info[2]
        pdb_file = pdb_id + ".pdb"
        chain_selection = "chain " + chain_id
        res_selection = "res " + res_id

        #command chains to process the pdb files.
        cmd.load(pdb_file)
        cmd.color("green")
        cmd.show("cartoon")
        cmd.hide("line")
        cmd.select("monomer", chain_selection)
        cmd.color("palecyan", "monomer")
        cmd.select("icp_dipep", chain_selection + " and " + res_selection)
        cmd.color("red", "icp_dipep")
        # cmd.zoom("monomer")
        # cmd.zoom("all")
        cmd.save(pdb_id + ".pse")
        #remove the previous work
        cmd.delete("all")
cmd.extend('mark_res', mark_res)

```


[1]:http://www.pymolwiki.org/index.php/Category:Script_Library
[2]:http://www.pymolwiki.org/index.php/Simple_Scripting
