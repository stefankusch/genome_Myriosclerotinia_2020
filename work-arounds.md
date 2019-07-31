This document contains a collection of work-arounds that I found helpful at some point.

1. Printing specific fasta sequences from a fasta file
2. Splitting a genome file into chromsome files
3. Sorting fasta file entries by header name
4. fastq or fq to fasta
5. GFF to fasta
6. (basic unix) Using `sed` to replace text in a file


#### Printing specific fasta sequences from a fasta file

One problem is that fasta sequences are often wrapped, i.e. there is a line break every 60 letters. I found this one-liner to remove the line breaks so that every sequence is in one line. 
```ShellSession
while read line;do if [ "${line:0:1}" '==' ">" ]; then echo -e "\n"$line; else echo $line | tr -d '\n' ; fi; done < 1980_CDS.fa > 1980_CDS.fasta
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
You can also run all this in one line, a bit messy but functions efficiently. The `grep -c` command at the end is to count fasta headers, which helps to assess whether the correct number of genes is in the new fasta file:
```ShellSession
grep -A1 -Fwf list_genes.txt sequences.fasta | awk '{gsub("-", "");print}' | grep -v -e '^$' > list_genes.fasta; grep -c '^>' list_genes.fasta
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

```Perl
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


#### Sorting fasta file entries by header name
I found the following `perl`/`sed` script to work nicely. 
```Perl
perl -pe 's/[\r\n]+/;/g; s/>/\n>/g' input-fasta.fa | sort -t"[" -k2,2V | sed 's/;/\n/g' | sed '/^$/d' > fasta_sorted.fa
```


#### .fastq/.fq to .fasta
A simple tool can be used to convert fastq files to fasta files:
```ShellSession
seqtk seq -a input.fastq > output.fasta
```


#### GFF to fasta using gff2fasta.pl
In order to acquire CDS, gene, exon, and upstream sequence fasta files from a genome and a GFF file, a Perl script `gff2fasta.pl` can be used. I downloaded the script from https://github.com/ISUgenomics/common_scripts/blob/master/gff2fasta.pl (save script with an editor like `nano`). To run the script within `bioconda` using `perl-bioperl` (needs to be installed), I used the following line: 
```ShellSession
perl -I ~/miniconda3/pkgs/perl-bioperl-1.6.924-4/lib/perl5/site_perl/5.22.0 PATH/gff2fasta.pl genome.fa annotation.gff output
```
The `-I PATH/perl-bioperl-1.6.924-4/lib/perl5/site_perl/5.22.0` function was required to direct perl to the `bioperl` functions. Outputs are 
```
output.cds.fasta #CDS sequences
output.cdna.fasta #cDNA sequences
output.exon.fasta #exon sequences
output.gene.fasta #full gene sequences
output.pep.fasta #protein sequences
output.upstream1000.fasta #1000 bp upstream sequences (in original script, 3000 bp)
```


#### Using `sed` to replace text in a file
This line directly replaces the text `Pattern` in the file. Does not write a new file.
```ShellSession
sed -i 's/PATTERN/REPLACEMENT/' file.name
``` 
To do that for all instances in the file:
```ShellSession
sed -i 's/PATTERN/REPLACEMENT/g' file.name
```

And to do that for a character string of all types of characters:
```ShellSession
sed 's/string.*//g' input.file > output.file
```
