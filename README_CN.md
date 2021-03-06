# 宏基因组有参分析流程

## 简介
此流程基于[Nextflow](https://www.nextflow.io/)实现。由新加坡基因组研究院([GIS](https://www.a-star.edu.sg/gis))计算及系统生物学第5组(CSB5)开发。

 - 使用Nextflow最新语法，流程模块化可重复使用
 - 尽可能提供Dockerfile和Conda YAML文件，运算环境可重复
 - 提供多种环境的配置文件，GIS高性能计算(SGE)，AWS (batch)，AWS Cluster (ignite)

## 依赖

### 主流程
 - [Nextflow](https://www.nextflow.io/)
 - Java Runtime Environment >= 1.8

### 质控和去宿主DNA
 - [Fastp](https://github.com/OpenGene/fastp) (>=0.20.0): 去接头
 - [BWA](https://github.com/lh3/bwa) (>=0.7.17): 去宿主DNA
 - [Samtools](https://github.com/samtools/samtools) (>=1.7): 去宿主DNA

### 有参宏基因组分析
 - [Kraken2](https://ccb.jhu.edu/software/kraken2/) (>=2.0.8-beta) + [Bracken](https://ccb.jhu.edu/software/bracken/) (>=2.5): 物种分类分析
 - [MetaPhlAn2](https://bitbucket.org/biobakery/metaphlan2/src/default/) (>=2.7.7): 物种分类分析
 - [HUMAnN2](https://bitbucket.org/biobakery/humann2/wiki/Home) (>=2.8.1): 代谢通路分析，以下脚本被修改（见[Running HUMAnN2 with reduced disk storage](docs/run_humann2.md)）：
   - `humann2.py`
   - `search/nucleotide.py`
 - [SRST2](https://github.com/katholt/srst2#installation) (=0.2.0): 抗药性分析

## 使用

在流程附带的数据上测试

```sh
$ shotgunmetagenomics-nf/main.nf -profile test
N E X T F L O W  ~  version 19.09.0-edge
Launching `./main.nf` [cheesy_volhard] - revision: dc7259a08e
WARN: DSL 2 IS AN EXPERIMENTAL FEATURE UNDER DEVELOPMENT -- SYNTAX MAY CHANGE IN FUTURE RELEASE
executor >  local (8)
[d4/2492b7] process > DECONT (SRR1950772)  [100%] 2 of 2 ✔
[3f/d7402d] process > KRAKEN2 (SRR1950772) [100%] 2 of 2 ✔
[de/a05395] process > BRACKEN (SRR1950772) [100%] 4 of 4 ✔
Completed at: 02-Oct-2019 16:21:34
Duration    : 3m 47s
CPU hours   : 0.5
Succeeded   : 8
```

显示全部帮助信息

```
$ shotgunmetagenomics-nf/main.nf --help

N E X T F L O W  ~  version 19.09.0-edge
Launching `shotgunmetagenomics-nf/main.nf` [fabulous_feynman] - revision: e8ec2a095b
WARN: DSL 2 IS AN EXPERIMENTAL FEATURE UNDER DEVELOPMENT -- SYNTAX MAY CHANGE IN FUTURE RELEASE
###############################################################################

      +++++++++++++++++'++++
      ++++++++++++++++''+'''
      ++++++++++++++'''''''+
      ++++++++++++++''+'++++
      ++++++++++++++''''++++
      +++++++++++++'++''++++
      ++++++++++++++++++++++       ++++++++:,   +++   ++++++++
      +++++++++++++, +++++++     +++.  .'+++;  +++  :+++   '++
      ++++++ ``'+`  ++++++++   +++'        ';  +++  +++      +
      ++++`   +++  +++++++++  +++              +++  +++:
      ++,  ,+++`  ++++++++++  ++;              +++    ++++
      +, ;+++  + .++++++++++  +++     .++++++  +++       ++++
      + `++;  ++  +++++;;+++  +++         +++  +++          '++,
      + :;   +++, ;++; ;++++  ;++;        +++  +++           +++
      +: ,+++++++;,;++++++++   `+++;      +++  +++  +.      ;++,
      ++++++++++++++++++++++      ++++++++++   +++   ++++++++.
===============================================================================
Usage:
The typical command for running the pipeline is as follows:
  nextflow run /mnt/projects/lich/stooldrug/scratch/shotgunmetagenomics-nf/main.nf  --read_path PATH_TO_READS

Input arguments:
  --read_path               Path to a folder containing all input fastq files (this will be recursively searched for *fastq.gz/*fq.gz/*fq/*fastq files) [Default: false]
  --reads                   Glob pattern to match reads, e.g. '/path/to/reads_*{R1,R2}.fq.gz' (this is in conflict with `--read_path`) [Default: false]
  
Output arguments:
  --outdir                  Output directory [Default: ./pipeline_output/]

Decontamination arguments:
  --decont_ref_path         Path to the host reference database
  --decont_index            BWA index prefix for the host

Profiler configuration:
  --profilers               Metagenomics profilers to run [Default: kraken2,metaphlan2,humann2]

Kraken2 arguments:
  --kraken2_index           Path to the kraken2 database

MetaPhlAn2 arguments:
  --metaphlan2_ref_path     Path to the metaphlan2 database
  --metaphlan2_index        Bowtie2 index prefix for the marker genes [Default: mpa_v20_m200]
  --metaphlan2_pkl          Python pickle file for marker genes [mpa_v20_m200.pkl]

HUMAnN2 arguments:
  --humann2_nucleotide      Path to humann2 chocophlan database
  --humann2_protein         Path to humann2 protein database

AWSBatch options:
  --awsqueue                The AWSBatch JobQueue that needs to be set when running on AWSBatch
  --awsregion               The AWS Region for your AWS Batch job to run on
###############################################################################
```

在GIS集群上使用

```sh
$ shotgunmetagenomics-nf/main.nf -profile gis --read_path PATH_TO_READS
```

使用Docker容器

```sh
$ shotgunmetagenomics-nf/main.nf -profile docker --read_path PATH_TO_READS
```

使用AWS batch （[AWS batch环境配置教程](https://t-neumann.github.io/pipelines/AWS-pipeline/)）

 - IAM配置 (在nextflow运行机器上设置环境变量AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION)
 - Batch 计算环境 & 作业队列
 - 定制AMI (AWS ECS optimized linux + 使用*miniconda*安装的awscli)

```sh
$ shotgunmetagenomics-nf/main.nf -profile test,awsbatch --awsqueue AWSBATCH_QUEUE --awsregion AWS_REGION -w S3_BUCKET --outdir S3_BUCKET 
```

支持提供多个profile, 例如: `-profile docker,test`.

同时运行多个分类器

```sh
$ shotgunmetagenomics-nf/main.nf -profile gis --profilers kraken2,metaphlan2 --read_path PATH_TO_READS
```

## 应用案例
 - Chng *et al*. Whole metagenome profiling reveals skin microbiome dependent susceptibility to atopic dermatitis flares. *Nature Microbiology* (2016)
 - Nandi *et al*. Gut microbiome recovery after antibiotic usage is mediated by specific bacterial species. *BioRxiv* (2018)
 - Chng *et al*. Cartography of opportunistic pathogens and antibiotic resistance genes in a tertiary hospital environment. *BioRxiv* (2019)


## 联系人
李陈浩：lichenhao.sg@gmail.com, lich@gis.a-star.edu.sg
