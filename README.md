# UMICollapse
Accelerating the deduplication and collapsing process for reads with Unique Molecular Identifiers (UMI). This tool implements many efficient algorithms for faster UMI deduplication. The preprint paper is available **[here](https://www.biorxiv.org/content/10.1101/648683v2)**. If you use this code, please cite

```
@article{liu2019algorithms,
  title={Algorithms for efficiently collapsing reads with Unique Molecular Identifiers},
  author={Liu, Daniel},
  journal={bioRxiv},
  year={2019},
  publisher={Cold Spring Harbor Laboratory}
}
```

## Installation
First, clone this repository:
```
git clone https://github.com/Daniel-Liu-c0deb0t/UMICollapse.git
cd UMICollapse
```
Then, install the dependencies, which are used for FASTQ/SAM/BAM input/output operations. Make sure you have Java 11.
```
mkdir lib
cd lib
curl -O -L https://repo1.maven.org/maven2/com/github/samtools/htsjdk/2.19.0/htsjdk-2.19.0.jar
curl -O -L https://repo1.maven.org/maven2/org/xerial/snappy/snappy-java/1.1.7.3/snappy-java-1.1.7.3.jar
cd ..
```
Now you have UMICollapse installed!

## Example Run
First, get some sample data from the UMI-tools repository. These aligned reads have their UMIs extracted and concatenated to their read headers. Make sure you have `samtools` installed to index the aligned BAM.
```
mkdir test
cd test
curl -O -L https://github.com/CGATOxford/UMI-tools/releases/download/1.0.0/example.bam
samtools index example.bam
cd ..
```
Finally, `test/example.bam` can be deduplicated.
```
./umicollapse bam -i test/example.bam -o test/dedup_example.bam
```
The UMI length will be autodetected, and the output `test/dedup_example.bam` should only contain reads that have a unique UMI. Unmapped reads are removed.

Here is an example with paired-end reads:
```
cd test
curl -O -L https://cibiv.github.io/trumicount/sg_100g.bam
samtools index sg_100g.bam
cd ..
```
Then, run
```
./umicollapse bam -i test/sg_100g.bam -o test/dedup_sg_100g.bam --umi-sep : --paired --two-pass
```

## Building
Run
```
./build.sh
```
to build the executable `.jar` file.

## Testing
Running basic tests after the `.jar` file is built:
```
./test.sh
```
There are also some small scripts for testing and debugging. For example, comparing two files to check if the UMIs are the same can be done with:
```
./run.sh test.CompareDedupUMI test/dedup_example_1.bam test/dedup_example_2.bam
```
or running benchmarks:
```
./run.sh test.BenchmarkTime 10000 10 1 ngrambktree
```

## Command-Line Arguments
### Mode (appears before commands)
* `sam` or `bam`: the input is an aligned SAM/BAM file with the UMIs in the read headers. This separately deduplicates each alignment coordinate.
* `fastq`: the input is a FASTQ file. This deduplicates the entire FASTQ file based on each entire read sequence.

### Commands
* `-i`: input file. Required.
* `-o`: output file. Required.
* `-k`: number of substitution edits to allow. Default: 1.
* `-u`: the UMI length. Default: autodetect.
* `-p`: threshold percentage for identifying adjacent UMIs in the directional algorithm. Default: 0.5.
* `-t`: parallelize the deduplication of each separate alignment position. Default: false.
* `-T`: parallelize the deduplication of one single alignment position. The data structure can only be `naive`, `bktree`, and `fenwickbktree`. Default: false.
* `--umi-sep`: separator string between the UMI and the rest of the read header. Default: `_`.
* `--algo`: deduplication algorithm. Either `cc` for connected components, `adj` for adjacency, or `dir` for directional. Default: `dir`.
* `--merge`: method for identifying which UMI to keep out of every two UMIs. Either `any`, `avgqual`, or `mapqual`. Default: `mapqual` for SAM/BAM mode, `avgqual` for FASTQ mode.
* `--data`: data structure used in deduplication. Either `naive`, `combo`, `ngram`, `delete`, `trie`, `bktree`, `sortbktree`, `ngrambktree`, `sortngrambktree`, or `fenwickbktree`. Default: `ngrambktree`.
* `--two-pass`: use a separate two-pass algorithm for SAM/BAM deduplication. This may be slightly slower, but it should use much less memory if the reads are approximately sorted by alignment coordinate. Default: false.
* `--paired`: use paired-end mode, which deduplicates pairs of reads. Currently removes chimeric alignments and unpaired reads. Default: false (single-end).

## Java Virtual Machine Memory
If you need more memory to process larger datasets, then modify the `umicollapse` file. `-Xms` represents the initial heap size, `-Xmx` represents the max heap size, and `-Xss` represents the stack size. If you do not know how much memory is needed, it may be a good idea to set a small initial heap size, and a very large max heap size, so the heap can grow when necessary. If memory usage is still is an issue, use the `--two-pass` option to save memory when the reads are approximately sorted (this is not a strict requirement, its just that when reads with the same alignment coordinate are close together in the file, they do not have to be kept in memory for very long).
