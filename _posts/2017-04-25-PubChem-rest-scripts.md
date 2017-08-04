---
layout: post
title:  lightweight shell script to annotate molfile output 
date:   2017-04-25
categories: PubChem, REST APIs, shell
---

I recently had a colleague ask for help examining the results of a structure
generation tool which dumps it output as a directory full of molfiles. He was
hoping to identify compounds in the set of commercial and scientific interest by
his strategy of manually searching database websites did not scale very well.
For my own use I might tap into the capabilities of RDKit but I wanted to set
him up with a simple system with minimal dependencies because I knew I'd be
completing my PhD and be less able to support him soon. Fortunately, the
combination of PubChem's [REST
                          API](https://pubchem.ncbi.nlm.nih.gov/pug_rest/PUG_REST.html) and ChemAxon's
simple but powerful [molconvert](https://docs.chemaxon.com/display/docs/Molecule+File+Conversion+with+MolConverter) command-line tool enabled me to to quickly
work up a shell script that summarized properties of each compound directory in
Excel readable .csv file and generated images of all of the compounds. 

```
#!/usr/bin/env/bash
echo "File,CIS,InChIKey,SMILES,Complexity,PubMed Count,Patent Count" >> $1/out.csv
for file in $(ls $1/*.mol); 
do
molconvert png:-H $file -o ${file[@]%.mol}.png
smiles=$(molconvert smiles:-H $file)
pt1=$(curl -gsf https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/smiles/"$smiles"/property/InChIKey,CanonicalSMILES,Complexity/csv|tail -1)
pt2=$(curl -gsf https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/smiles/"$smiles"/xrefs/PubMedID/txt|wc -l)
pt3=$(curl -gsf https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/smiles/"$smiles"/xrefs/PatentID/txt|wc -l)
echo $file,$pt1,$pt2,$pt3 >> $1/out.csv
done
```
