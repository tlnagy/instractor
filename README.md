## instractor

Instractor is a simple tool written in Julia to extract desired target DNA sequence (including targets flanked by variable length up/down-stream sequence) from Paired End (PE) Illumina reads. It's able to take overlapping sequence quality scores into consideration and doesn't care about target length. It also offers a few different ways to write output, including a human friendly visualization format.

This tool is similar to more sophisticated and better validated tools like [PEAR](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3933873/), iTag, BIPES, Shera, FLASH, PANDAseq and COPE, so you should check out these projects if instractor doesn't do what you'd like. Alternatively, feel free to add a "enhancement" feature request as an issue, and I'd be happy to take it into consideration. 

## Problem

It is often desirable to sequence DNA inserts that have been cloned into some vector using Illumina technology. Getting the sequence of just the inserted region is often all that is needed and can be accomplished with custom read primers. However, on occasion, it is not possible to design custom read primers close enough to the insert, meaning a bit of vector sequence has to be sequenced to get to the insert. This could even be desired if one wanted to check that cloning worked as planned! 

Unfortunately, this situation poses a problem for Illumina sequencing machines. In order to focus the camera and identify clusters, reads must high complexity in the first several base positions (see [this](https://sequencing.qcfail.com/articles/biased-sequence-composition-can-lead-to-poor-quality-data-on-illumina-sequencers/) if you don't know what I'm talking about).

Another related issue that commonly arises is that the first and last several base calls of a read tend to be of lower quality making it beneficial for important base calls to begin a little further into a read.

One solution is to spike in a lot of PhiX, but this is wasted sequencing capacity! Another solution is to create offset library index primers, essentially shifting read starts a few base pairs in either direction to increase complexity. 

This second solution creates reads that are all offset in position relative to each other. Additionally, there is still vector DNA to cleave from one (or both) ends of the insert. 

This software is a bioinformatic solution to both of these problems. Additionally, it processes overlapping reads, creating a new read that merges base calls, keeping the better quality base (and score) when ambiguities exist. 


<br/>


## Quickstart: Installing
The software requires Julia 1.1.0 (or later, but untested). An installation guide is here:  
[Platform specific instructions for installing Julia](https://julialang.org/downloads/platform.html)

Get the code:
  
  
```bash
git clone https://github.com/ccario83/instractor.git
```

You\'ll also need a couple Julia packages:


```bash
julia -e 'import Pkg; Pkg.add("ArgParse"); Pkg.add("GZip")'
```

<br/>

## Quickstart: Running

Check out the script parameters with:

```bash
./instractor.jl --help
```

<br/>  

### Example 1: Simply merge reads based on best overlap 

The script requires at minimum two read files, specified with `--read1` and `--read2` respectively. Read files can be either ***.fastq** or ***.fastq.gz** files. You should also specify an output file for the default mode (more later on modes):

```bash
./instractor.jl --read1 examples/R1_notrim.fastq --read2 examples/R2_notrim.fastq -o examples/example1_output.fastq
```
<br/>

Screen output:
```bash
🧬  Summary
Total processed:     	    1001
# errors:            	      15
# successful:        	     986
Successfully parsed: 	   98.50%

🧬  Errors
Read alignment:     	      14
Insert size:        	       0
No insert:          	       0

Wrote "examples/example1_output.fastq" 
```

<br/>
<br/>
You'll also see the output reads in fastq format (read1's header with the insert alignment score is appended to the header line): 

*examples/example1_output.fastq:*

```bash
@NS500221:298:HCFTVBGXB:1:11101:5483:1102 1:N:0:TAGCGCTC+AGGCTTNG  0.810  
ATGGGCAACTATAATGGGCAGAATACGGCTTCGCTAAGTGTATTCATCCCCCCCTACTTCGCGGAGAAGATCATACTTACAGAGATGCCTTGTTCGACAGATACAAAC  
+  
AAAAAEEEEEEEEEE6EEEEEEE6EAEAE/EEEEEEEEAAEEAEE6AEE<AAE<E6E/AEEEEEEE6EAAEEE<E/A#//EE/EE#EE/##EE#EEEEEEE#E////A  
@NS500221:298:HCFTVBGXB:1:11101:13566:1105 1:N:0:TAGCGCTC+AGGCTTNG  0.814  
GTGTCACTAGCGCGTCGGAACTCCAGCGAGCGCACACTTTCCCCAGGATTGCCTAGCGGGTCGATGTCACCGCTACATACTCCACTACATTCCTCCCTCGTTTCATTT  
+  
AAAAAEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEAEEEEEEEEEEEEEEAEEEEEEEE#EEEEEEE#EEE##EE#EEEEEEE#EAAAAA  
@NS500221:298:HCFTVBGXB:1:11101:6317:1109 1:N:0:TAGCGCTC+AGGCTTNG  0.798  
GAGGCGAAAGACGAATGCCGAAGCGCCATGGAAGCTCTCAAACAAAAGAGTCTTTATAACTGTCGATGTAAAAGGGGTATGAAAATGGATTAGTATTGTCTTCGCATA  
```

<br/>
<br/>

### Example 2: Specifying expected insert lengths 

Specifying expected insert length will filter out reads that don't meet this expectation, specify this number with `-e`

```bash
./instractor.jl -e 108 --read1 examples/R1_notrim.fastq --read2 examples/R2_notrim.fastq -o examples/example2_output.fastq
```
<br/>
<br/>
Screen output:

```bash
🧬  Summary
Total processed:     	    1001
# errors:            	     487
# successful:        	     514
Successfully parsed: 	   51.35%

🧬  Errors
Read alignment:     	      14
Insert size:        	     472
No insert:          	       0

Wrote "examples/example2_output.fastq"
```

<br/>
<br/>

### Example 3: Visualizing alignment

In addition to writing an output file, you can visualize how the alignment is actually going by changing the mode to 'show' with `-m`:

```bash
./instractor.jl -e 108 -m show --read1 examples/R1_notrim.fastq --read2 examples/R2_notrim.fastq -o examples/example3_output.fastq
```

Now you'll see entries like this printed to the screen:

```
Entry: 1000   
>>>
▅▅▅▅▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆
AACACGAGTCAAAGTCAAAAGAAAGGACAGCAATCCCAGTTTTTACAGAGCAGGAACCTGAAATGCATCGCGTTGT
                                ATCCCAGTTTTTACAGAGCAGGAACCTGAAATGCATCGCGTTGTGCGAAGTCATAACCTCTTCGGATCTACATAAG
                                ▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▅▅▅▅▅


▅▅▅▅▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▅▅▅▅▅
AACACGAGTCAAAGTCAAAAGAAAGGACAGCAATCCCAGTTTTTACAGAGCAGGAACCTGAAATGCATCGCGTTGTGCGAAGTCATAACCTCTTCGGATCTACATAAG
Length:  108
Offset:    0

Alignment score: 1.00
<<<
```

Here we see quality scores represented as vertical bars (the taller the better!), read 1's sequence, read 2's reverse complemented sequence and quality scores, and their overlap. The final molecule's quality, sequence, and length is also shown. Final base calls and quality scores are simply taken from the original read, on in the case where there is overlap from the read with the better quality.

<br/>
<br/>

### Example 4: Trimming vector sequence 

Let's say we've used offset primers for sequencing and have a bit of flanking vector sequence to trim before writing the insert. This can be done with the `-L` and `-F` flags for leading and following vector sequence, respectively: 

```bash
./instractor.jl -e 21 -m show --read1 examples/R1_trim.fastq.gz --read2 examples/R2_trim.fastq.gz -L CGCAATTCCTTTAGTGGTACCTTTCTATTCTCACTCT -F CTTTCAACAGTTTCGGCCGAACCTCCACC -o examples/example4_output.fastq
```

Now you\'ll see two additional sequences that correspond to the input leading and following sequences, as well as information about their alignment. The insert sequence is now the sequence between these. A '~' symbol is used to show where these leading/following sequences offset from the read:

```
Entry: 5437   
>>>
CGCAATTCCTTTAGTGGTACCTTTCTATTCTCACTCT
▅▅▅▅▅▅▄▄▆▆▆▆▄▄▆▅▅▅▆▆▆▄▄▆▆▆▅▄▄▄▆▆▆▄▄▄▄▅▆▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▄▄▄▅▆▄▆▆▄▄▃▅▅▄
CGCAATTCCTTTAGTGCTACCTTTCGAGGCTCACTCTGATATGTCTCTTCATATTAGTGGTGGAGGTTCGGAAGAA
              TGGTACCTTTCTATGCTCACTCTGATATGTCTCTTCATATTAGTGGAGGAGGTTCGGACGAAACTGTAGAAAGCCT
              ▃▆▃▅▆▄▄▄▃▆▅▆▆▄▃▅▅▆▆▄▆▄▄▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▄▄▄▃▃▅▄▃▃▄▆▅▃▄▄▄▅▆▆▄▄▄▃▄▄▆▃▅▅▅▅▅
                                                          GGTGGAGGTTCGGCCGAAACTGTTGAAAG~~~

                                     ▅▆▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆
Extracted insert:                    GATATGTCTCTTCATATTAGT
Length:   21
Offset:   37

Alignment score: 0.92
Leader score:    0.95
Follower score:  0.93
<<<
```
<br/>
<br/>

### Example 5: Writing output as compressed fastq.gz 

If downstream analysis is able to handle compressed fastq files, it may be desirable to save disk space and write them directly. To do so, simply change the `--format` argument to "fastq.gz": 

```bash
 ./instractor.jl --read1 examples/R1_notrim.fastq --read2 examples/R2_notrim.fastq --format "fastq.gz" -o examples/example5_output.fastq.gz
```

<br/>
<br/>


### Example 6: Writing output as fasta or compressed fasta.gz 

Instractor also supports fasta/fasta.gz file formats with the `--format` argument. Use "fasta" or "fastq.gz" to specify: 

```bash
 ./instractor.jl --read1 examples/R1_notrim.fastq --read2 examples/R2_notrim.fastq --format "fasta.gz" -o examples/example6_output.fasta.gz
```

<br/>
<br/>


### Example 7: Print options in 'show' mode

There are a few different ways to control what sequence information is shown in 'show' mode. By default, the read alignment, leader/follower, and insert sequences are shown. You can also display the consensus sequence using `-C`. If you'd prefer to not show the read alignment, you can specify `-R`. The leader/follower sequences will then be shown on the consensus sequence in this case if `-C` is specified. Finally, to suppress showing the insert, you can use `-I`. In example 7, we choose to suppress the read alignment and show the consensus instead, while also keeping the insert sequence (by adding `-C -R` to the command): 

```bash
./instractor.jl -e 21 -m show -C -R --read1 examples/R1_trim.fastq.gz --read2 examples/R2_trim.fastq.gz -L CGCAATTCCTTTAGTGGTACCTTTCTATTCTCACTCT -F CTTTCAACAGTTTCGGCCGAACCTCCACC -o examples/example7_output.fastq 
```

You can compare the resulting output below to example 4 to see the difference.

```
Entry: 5437   
>>>
Consensus:
CGCAATTCCTTTAGTGGTACCTTTCTATTCTCACTCT
▅▅▅▅▅▅▄▄▆▆▆▆▄▄▆▅▅▅▆▆▆▄▄▆▆▆▅▄▄▄▆▆▆▄▄▄▄▅▆▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▄▄▄▅▆▄▆▆▄▄▄▅▅▄▆▄▄▄▃▄▄▆▃▅▅▅▅▅
CGCAATTCCTTTAGTGCTACCTTTCTATGCTCACTCTGATATGTCTCTTCATATTAGTGGTGGAGGTTCGGACGAAACTGTAGAAAGCCT
                                                          GGTGGAGGTTCGGCCGAAACTGTTGAAAG~~~

Extracted insert:
                                     ▅▆▅▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆▆
                                     GATATGTCTCTTCATATTAGT

Length:   21
Offset:   37
Alignment score: 0.92
Leader score:    0.95
Follower score:  0.93
<<<
```

<br/>
<br/>

## Run Modes

There are several modes in which to run this script, as specified from the `-m` argument. We've seen a couple, here is a complete list:


1) `none` -- Don't write anything to STDOUT, just write the output file
2) `summary` -- Write summary info and the output file
3) `stream` -- Stream the output instead of writing it to a file
4) `show` -- Show the parsed alignments and quality scores  

Additionally, instead of writing to **fasta/fastq**, read information can be written to a **tsv** file by specifying `-f tsv` when running the script


**Questions/Concerns**
Feel free to reach out with questions, concerns or ideas. I'm also happy to consider feature requests; please submit as issues.
