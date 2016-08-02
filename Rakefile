require 'rake/clean'
require 'yaml'

CLOBBER.include(:default)

DIR_FASTA_SPLIT = "fasta_split"
directory DIR_FASTA_SPLIT

DIR_BATCH_SCRIPTS = "batch_scripts"
directory DIR_BATCH_SCRIPTS

DIR_SGE_LOGS = "batch_logs"
directory DIR_SGE_LOGS

BATCH_SCRIPT_TEMPLATE = "run_iprscan_batch.template.sh"


## load conf
$conf = YAML.load_file("conf.yml")
p $conf

query = $conf['query']
num_fasta_per_subfile = $conf['num_fasta_per_subfile']

desc "build the template of batch script from conf"
task :build_batch_template do |t|
  $conf['batch_script_template']
  temp = File.open($conf['batch_script_template']).read
  txt = File.open($conf['batch_script_template_sge']).read

  temp.sub!(/%SGE_PART%/, txt)
  temp.sub!(/%INTERPROSCAN_DIR%/, $conf['interproscan_dir'])

  File.open(BATCH_SCRIPT_TEMPLATE, "w"){|o| o.puts temp}
  puts "Template for batch jobs was generated: #{BATCH_SCRIPT_TEMPLATE}"
end


desc "split query fasta files"
task :split_query do
  puts "split_query"

  sh "ruby util/split_and_clean_fasta.rb #{query} #{num_fasta_per_subfile} #{DIR_FASTA_SPLIT}"
end


desc "generate scripts for batch interproscan"
file :generate_batch_jobs => FileList["#{DIR_FASTA_SPLIT}/*[0-9].fasta"] do |t|
  target_list = File.basename(BATCH_SCRIPT_TEMPLATE, ".template.sh") + ".list"
  sh "touch #{target_list}"
  sh "echo '#FASTAF' > #{target_list}"
  sh "ls -1 #{DIR_FASTA_SPLIT}/*[0-9].fasta >> #{target_list}"
  sh "ruby util/generate_batch_jobs.rb #{BATCH_SCRIPT_TEMPLATE} #{target_list} #{DIR_BATCH_SCRIPTS}"
end

desc "submit job scripts to SGE"
task :sge_submit_jobs do
  FileList["#{DIR_BATCH_SCRIPTS}/*.job*.sh"].each do |f|
    sh "qsub -V #{f}"
  end
end


desc "SGE result checking"
task :postchk_sge => DIR_SGE_LOGS do |t|
  queries = []
  File.open("run_iprscan_batch.list").each do |l|
    next if /^\#/.match(l)
    queries << l.chomp.split(/\//)[-1]
  end

  queries.each do |q|
    unless File.exists?("#{q}.iprscan.finished")
      raise "ERROR: the following job has not been completed. \n#{q} "
    end
  end

  FileList["#{DIR_BATCH_SCRIPTS}/*.job*.sh"].each do |f|
    logs = FileList["#{File.basename(f)}.*"]
    sh "mv #{logs.join(' ')}  #{DIR_SGE_LOGS}"
  end
end


desc "combine batch results into a single file"
task :combine_blast_results do
  queries = []
  File.open("run_iprscan_batch.list").each do |l|
    next if /^\#/.match(l)
    queries << l.chomp.split(/\//)[-1]
  end

p queries

  queries_sorted = queries.sort{|a, b|
    ma = /_(\d+)\.fasta$/.match(a)
    num_a = ma[1].to_i
    mb = /_(\d+)\.fasta$/.match(b)
    num_b = ma[1].to_i
    num_a <=> num_b
  }
  p queries_sorted

  puts "#{queries_sorted.size} queries."

  outf_xml = queries_sorted[0].dup.sub(/_1.fasta/, '.iprscan.xml')
  outf_tsv = queries_sorted[0].dup.sub(/_1.fasta/, '.iprscan.tsv')

  o_xml = File.open(outf_xml, "w")
  o_tsv = File.open(outf_tsv, "w")

  queries_sorted.each do |q|
    xml = "#{q}.xml"
    tsv = "#{q}.tsv"
    o_xml.puts File.open(xml).read
    o_tsv.puts File.open(tsv).read
  end

  puts "saved in #{outf_xml}"
  puts "saved in #{outf_tsv}"

end


#test_pep.fasta_6.fasta.iprscan.finished
#test_pep.fasta_6.fasta.tsv
#test_pep.fasta_6.fasta.xml

# NOT IMPLEMENTED YET
desc "Final check and cleaning"
task :finalize do |t|

#  unless blast_results_combined
#    raise "combined blast result file has not been generated."
#  end

  queries = []
  File.open("run_iprscan_batch.list").each do |l|
    next if /^\#/.match(l)
    queries << l.chomp.split(/\//)[-1]
  end

p queries
  queries.each do |q|
  p q
    sh "rm #{q}.tsv"
    sh "rm #{q}.xml"
    sh "rm #{q}.iprscan.finished"
  end


end
