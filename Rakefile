require "rake/testtask"

spec = Gem::Specification.load(File.expand_path("vanity.gemspec", File.dirname(__FILE__)))

desc "Push new release to gemcutter and git tag"
task :push do
  sh "git push"
  puts "Tagging version #{spec.version} .."
  sh "git tag #{spec.version}"
  sh "git push --tag"
  puts "Building and pushing gem .."
  sh "gem build #{spec.name}.gemspec"
  sh "gem push #{spec.name}-#{spec.version}.gem"
end

desc "Install #{spec.name} locally"
task :install do
  sh "gem build #{spec.name}.gemspec"
  sudo = "sudo" unless File.writable?( Gem::ConfigMap[:bindir])
  sh "#{sudo} gem install #{spec.name}-#{spec.version}.gem"
end


task :default=>:test
desc "Run all tests (also default task)"
Rake::TestTask.new do |task|
  task.test_files = FileList['test/*_test.rb']
  if Rake.application.options.trace
    #task.warning = true
    task.verbose = true
  elsif Rake.application.options.silent
    task.ruby_opts << "-W0"
  else
    task.verbose = true
  end
end

task(:clobber) { rm_rf "tmp" }


begin
  require "yard"
  YARD::Rake::YardocTask.new(:yardoc) do |task|
    task.files  = FileList["lib/**/*.rb"].exclude("lib/vanity/backport.rb")
    task.options = "--output", "html/api", "--title", "Vanity #{spec.version}", "--main", "README.rdoc", "--files", "CHANGELOG"
  end
rescue LoadError
end

desc "Jekyll generates the main documentation (sans API)"
task(:jekyll) { sh "jekyll", "doc", "html" }

desc "Create documentation in docs directory (including API)"
task :docs=>[:jekyll, :yardoc]
desc "Remove temporary files and directories"
task(:clobber) { rm_rf "html" }

desc "Publish documentation to vanity.labnotes.org"
task :publish=>[:clobber, :docs] do
  sh "rsync -cr --del --progress html/ labnotes.org:/var/www/vanity/"
end


task :report do
  $LOAD_PATH.unshift "lib"
  require "vanity"
  require "timecop"
  Vanity.playground.load_path = "test/experiments"
  Vanity.playground.experiments.each(&:destroy)
  Vanity.playground.metrics.values.each(&:destroy!)
  Vanity.playground.reload!

  # Control	182	35	19.23%	N/A
  # Treatment A	180	45	25.00%	1.33
  # Treatment B	189	28	14.81%	-1.13
  # Treatment C	188	61	32.45%	2.94
  experiment(:null_abc).fake nil=>[182,35], :red=>[180,45], :green=>[189,28], :blue=>[188,61]

  experiment(:age_and_zipcode).fake false=>[80,35], true=>[84,32]

  cheers, yawns = 0, 0
  (Date.today - 80..Date.today).each do |date|
    Timecop.travel date do
      cheers = cheers - 5 + rand(20)
      Vanity.playground.track! :yawns, cheers
      yawns = yawns - 5 + rand(30)
      Vanity.playground.track! :cheers, yawns
    end
  end

  Vanity::Commands.report ENV["OUTPUT"]
end
