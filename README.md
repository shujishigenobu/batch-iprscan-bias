batch-iprscan-bias
================

A tool for batch InterProScan search. It is optimized for use on NIBB bias4 server.

## Requrement

*  ruby
*  InterProScan (>= v5.19)

## Quick Start

### 1) Download software

Download `batch-iprscan-bias` package from GitHub (https://github.com/shujishigenobu/batch-iprscan-bias)

```bash
$ git clone git@github.com:shujishigenobu/batch-iprscan-bias.git
```

### 2) Prepare coniguration file, conf.yml

`batch-iprscan-bias` requires `conf.yml`, a configuration file. An example file is included in the package. It would be a good idea to create your own `conf.yml` from `conf.yml.example` by copying.

```bash
$ cp conf.yml.example conf.yml
```

Edit `conf.yml` using your favorite editor. An example is as follows:

```yaml
query: test.pep.fa
num_fasta_per_subfile: 100

interproscan_dir:  ~/bio/applications/interproscan-5.19-58.0/

batch_script_template:     templates/run_interproscan.sh.template
batch_script_template_sge: templates/sge.bias.small
```

### 3) 4 steps to start batch jobs

```bash
$  rake build_batch_template
```
edit `build_batch_template` if needed.

```bash
$  rake split_query
$  rake generate_batch_jobs
$  rake sge_submit_jobs
```
Now your jobs are queued. Monitor your jobs by SGE command like `qstat`.

### 4) 3 steps for combine InterProScan results

not implemented yet.



## License

Copyright 2016 Shuji Shigenobu <shige@nibb.ac.jp>.

Licensed under the MIT License
