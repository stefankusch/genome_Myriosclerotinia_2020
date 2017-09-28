This document contains a collection of work-arounds that I found helpful at some point.

1. Printing specific fasta sequences from a fasta file
2. Splitting a genome file into chromsome files
3. 


#### Printing specific fasta sequences from a fasta file

One problem is that fasta sequences are often wrapped, i.e. there is a line break every 60 letters. I found this one-liner to remove the line breaks so that every sequence is in one line. 
```ShellSession
while read line;do if [ "${line:0:1}" == ">" ]; then echo -e "\n"$line; else echo $line | tr -d '\n' ; fi; done < 1980_CDS.fa > 1980_CDS.fasta
```

Then, you can print the sequences of the genes you are interested in to a new file using `grep`.
```ShellSession
grep -A1 -f list.genes-of-interest.txt 1980_CDS.fasta > subset.sequences.fasta
```

If you look at the file, you see that there is a line containing `--` between every fasta entry now. To remove that, do
```ShellSession
awk '{gsub("-", "");print}' subset.sequences.fasta > sequences.fasta
```
And, to remove the empty lines, use `grep` again:
```ShellSession
grep -v -e '^$' sequences.fasta > sequences.final.fasta
```
You can also run all this in one line, a bit messy but functions efficiently:
```ShellSession
grep -A1 -F -f list.genes-of-interest.txt sequences.fasta > tmp1.fasta; awk '{gsub("-", "");print}' tmp1.fasta > tmp1.out; grep -v -e '^$' tmp1.out > genes-of-interest.fasta; rm tmp1.out tmp1.fasta; head genes-of-interest.fasta; head list.genes-of-interest.txt
```

Note: Mind that in case of numbering of contigs, scaffolds, etc, grep will find all occurences. Thus, do not name 
```
contig_1
contig_2
...
```
but rather
```
contig_0001
contig_0002
...
contig_0805
```



#### Splitting a genome file into chromsome files

This perl-script will do the job. Copy the script in a file (e.g. with nano or editor) and call it `split.pl`.

```perl6
#!/usr/bin/perl

$f = $ARGV[0]; #get the file name

open (INFILE, "<$f")
or die "Can't open: $f $!";

while (<INFILE>) {
$line = $_;
chomp $line;
if ($line =~ /\>/) { #if has fasta >
close OUTFILE;
$new_file = substr($line,1);
$new_file .= ".fa";
open (OUTFILE, ">$new_file")
or die "Can't open: $new_file $!";
}
print OUTFILE "$line\n";
}
close OUTFILE;
```

Run it with 
```ShellSession
perl PATH/split.pl PATH/genome.fasta 
``` 
after navigating into the directory where the chromosome files are supposed to be stored. 
