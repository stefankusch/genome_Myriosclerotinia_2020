This document contains a collection of work-arounds that I found helpful at some point.

1. Printing specific fasta sequences from a fasta file
2. 


#### Printing specific fasta sequences from a fasta file

One problem is that fasta sequences are often wrapped, i.e. there is a line break every 60 letters. I found this one-liner to remove the line breaks so that every sequence is in one line. 
```
while read line;do if [ "${line:0:1}" == ">" ]; then echo -e "\n"$line; else echo $line | tr -d '\n' ; fi; done < 1980_CDS.fa > 1980_CDS.fasta
```

Then, you can print the sequences of the genes you are interested in to a new file using `grep`.
```
grep -A1 -f list.genes-of-interest.txt 1980_CDS.fasta > subset.sequences.fasta
```

If you look at the file, you see that there is a line containing `--` between every fasta entry now. To remove that, do
```
awk '{gsub("-", "");print}' subset.sequences.fasta > sequences.fasta
```

