﻿# kmcEx
The k-mers along with their frequency have been served as the elementary building block for error correction, repeat detection, multiple sequence alignment, genome assembly, etc, attracting intensive studies in k-mer counting. However, the result of k-mer counters itself is pretty large; very often, it is too large to ﬁt into the main memory, which in turn signiﬁcantly narrows down its usability. To overcome this bottleneck, we introduce a very novel idea of encoding k-mers as well as their frequency achieving ultra memory saving and retrieval eﬃcient. Precisely, we propose a Bloom Filter-like data structure to encode kmers as well as their frequency by coupling-bit arrays— one for k-mer representation and the other for frequency encoding. Experimental results conducted on ﬁve real data sets show that the compression ratio is as high as 7.8 via encoding. Besides, we have achieved constant time complexity in retrieving a k-mer as well as its frequency and two magnitudes smaller false positive rate.

# installation 
kmcEx is based on **C++11**，and the installation step is very simple, in the kmodel main directory run 'make'  command, then you can get the executable kmodel.
```
make # run in the main directory 
```
# usage
how to use  the executable kmcEx to test
```
1. USAGE
     kmcEx [options] <input_file_name> <output_file_name> <working_directory>
     kmcEx [options] <@input_file_names> <output_file_name> <working_directory>
2. OPTIONS
	1) REQUIRED
		input_file_name - single file in FASTQ format (gziped or not)
		@input_file_names - file name with list of input files in FASTQ format (gziped or not)
		working_directory -save some temporary files
	2) OPTIONAL
		-k<len> - k-mer length (k<=31; default: 25)
		-t<value> - total number of threads (default: 4)
		-ci<value> - exclude k-mers occurring less than <value> times (default: 2)
		-cx<value> - exclude k-mers occurring more of than <value> times (default: 1000)
		-cs<value> - maximal value of a counter(default: 1000)
		-nh<value> - number of hash (default: 7)
		-nb<value> - number of bit array (default: 4)
		-test<value> -test the model (0 or 1 default: 0)
3. EXAMPLES
	kmcEx -k27 -nh7 -nb4  rs.fastq rs.res /tmp
	kmcEx -k27 -nh7 -nb4  @rs.lst rs.res /tmp
```

# use demo

```
#!/bin/bash
kn=31
out_dir=/tmp 
ci=2
cs=1000
cx=1000
./kmcEx -ci${ci} -cs${cs} -cx${cx} -k${kn} -t8 -m24 @sa.lst $out_dir/SA_k${kn}_f${ci}-${cx}.res /tmp
./kmcEx -ci${ci} -cs${cs} -cx${cx} -k${kn} -t8 -m24 @rs.lst $out_dir/RS_k${kn}_f${ci}-${cx}.res /tmp
./kmcEx -ci${ci} -cs${cs} -cx${cx} -k${kn} -t8 -m24 @hc14.lst $out_dir/HC14_k${kn}_f${ci}-${cx}.res /tmp
./kmcEx -ci${ci} -cs${cs} -cx${cx} -k${kn} -t8 -m24 @bi.lst $out_dir/BI_k${kn}_f${ci}-${cx}.res /tmp
./kmcEx -ci${ci} -cs${cs} -cx${cx} -k${kn} -t8 -m24 @na12878_1.lst $out_dir/NA12878_k${kn}_f${ci}-${cx}.res /tmp
```
the file **sa.lst**  is the location of fastq, one line one location, like this:
```
/share/share/data/NGS/GAGE/RS/rs_frag_1.fastq
/share/share/data/NGS/GAGE/RS/rs_frag_2.fastq
```
after running the command, in the $out_dir，you will get kmc_database file:

    RS_k31_f2-1000.res.kmc_pre
    RS_k31_f2-1000.res.kmc_suf
the model saved  in  /tmp/RS_k31_f2-1000.res  (working_directory)，and there are some files like this:

    bit1.bin  bit2.bin  bloom.bin  hash.bin  last_map.bin  occ.bin  param.conf

# API usage
create a kmodel and save it to the disk，since the number of kmer(k=1) is very huge，we distinguish this two situation that k>=2 and k>=1 
1) for k>=2
```c
	//some parameters
	int n_hash = 7, n_bit = 4;
	//kmc_database is the kmc database file,such as 'RS_k31_f2-1000.res'
	string kmc_database = "/tmp/RS_k31_f2-1000.res";
	OccuBin* occu_bin = new OccuBin(n_hash);
	KModel* kmodel = new KModel(occu_bin, n_bit);
	//initialize the model
	kmodel->init_KModel(kmc_database);
	//save model to an existing directory "model_dir",
	kmodel->save_model("/tmp/rs_f2-1000_model");
```
2) for k>=1
```c
	int n_hash = 7, n_bit = 4;
	//kmc_database is the kmc database file,such as 'RS_k31_f2-1000.res'
	string kmc_database = "/tmp/RS_k31_f1-1000.res";
	OccuBin* occu_bin = new OccuBin(n_hash);
	//KModelOne is a subclass of Kmodel
	KModel* kmodel = new KModelOne(occu_bin, n_bit);
	//initialize the model
	kmodel->init_KModel(kmc_database);
	//save model to an existing directory "model_dir",
	kmodel->save_model("/tmp/rs_f1-1000_model");
```

3) load a model from  the disk, and get the occurrence of some kmer
```c
#include "kmodel.hpp"
KModel* kmodel = new KModel();
kmodel->load_model(model_dir); //load the model from the disk
//query one kmer 
string kmer=...;
int occ=kmodel->kmer_to_occ(kmer) 
//query multiple kmer with multiple threads
vector<string> kmer_v=...;
kmodel->kmer_to_occ(kmer_v);
```