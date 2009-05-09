require 'rake'
require 'rake/packagetask'

SCRIPTY2_ROOT     = File.expand_path(File.dirname(__FILE__))
SCRIPTY2_SRC_DIR  = File.join(SCRIPTY2_ROOT, 'src')
SCRIPTY2_DIST_DIR = File.join(SCRIPTY2_ROOT, 'dist')
SCRIPTY2_DOC_DIR  = File.join(SCRIPTY2_ROOT, 'doc')
SCRIPTY2_PKG_DIR  = File.join(SCRIPTY2_ROOT, 'pkg')
SCRIPTY2_VERSION  = YAML.load(IO.read(File.join(SCRIPTY2_SRC_DIR, 'constants.yml')))['SCRIPTY2_VERSION']

SCRIPTY2_TEST_DIR      = File.join(SCRIPTY2_ROOT, 'test')
SCRIPTY2_TEST_UNIT_DIR = File.join(SCRIPTY2_TEST_DIR, 'unit')
SCRIPTY2_TMP_DIR       = File.join(SCRIPTY2_TEST_UNIT_DIR, 'tmp')

$:.unshift File.join(SCRIPTY2_ROOT, 'vendor', 'sprockets', 'lib')

def sprocketize(path, source, destination = source)
  begin
    require "sprockets"
  rescue LoadError => e
    puts "\nYou'll need Sprockets to build script.aculo.us 2. Just run:\n\n"
    puts "  $ git submodule init"
    puts "  $ git submodule update"
    puts "\nand you should be all set.\n\n"
  end
  
  secretary = Sprockets::Secretary.new(
    :root         => File.join(SCRIPTY2_ROOT, path),
    :load_path    => [SCRIPTY2_SRC_DIR],
    :source_files => [source]
  )
  
  secretary.concatenation.save_to(File.join(SCRIPTY2_DIST_DIR, destination))
end

task :default => [:dist, :package, :clean_package_source]

desc "Builds the distribution."
task :dist do
  sprocketize("src", "s2.js")
end

namespace :doc do
  desc "Builds the documentation."
  task :build => [:require] do
    
    TEMPLATES_ROOT = File.join(SCRIPTY2_ROOT, "vendor", "pdoc",
      "new_templates")
    
    TEMPLATES_DIRECTORY = File.join(TEMPLATES_ROOT, "html")
    
    require 'tempfile'
    begin
      require "sprockets"
    rescue LoadError => e
      puts "\nYou'll need Sprockets to build script.aculo.us 2. Just run:\n\n"
      puts "  $ git submodule init"
      puts "  $ git submodule update"
      puts "\nand you should be all set.\n\n"
    end
    
    Tempfile.open("pdoc") do |temp|
      secretary = Sprockets::Secretary.new(
        :root           => File.join(SCRIPTY2_ROOT, "src"),
        :load_path      => [SCRIPTY2_SRC_DIR],
        :source_files   => ["s2.js"],
        :strip_comments => false
      )
        
      secretary.concatenation.save_to(temp.path)
      rm_rf SCRIPTY2_DOC_DIR
      PDoc::Runner.new(temp.path, {
        :output    => SCRIPTY2_DOC_DIR,
        :templates => File.join(TEMPLATES_DIRECTORY, "html")
      }).run
    end
  end  
  
  task :require do
    lib = 'vendor/pdoc/lib/pdoc'
    unless File.exists?(lib)
      puts "\nYou'll need PDoc to generate the documentation. Just run:\n\n"
      puts "  $ git submodule init"
      puts "  $ git submodule update"
      puts "\nand you should be all set.\n\n"
    end
    require lib
  end
end

task :doc => ['doc:build']

Rake::PackageTask.new('scripty2', SCRIPTY2_VERSION) do |package|
  package.need_tar_gz = true
  package.package_dir = SCRIPTY2_PKG_DIR
  package.package_files.include(
    'README.rdoc',
    'MIT-LICENSE',
    'dist/s2.js',
    'src/**'
  )
end

task :clean_package_source do
  rm_rf File.join(SCRIPTY2_PKG_DIR, "scripty2-#{SCRIPTY2_VERSION}")
end

task :test => ['test:build', 'test:run']
namespace :test do
  desc 'Runs all the JavaScript unit tests and collects the results'
  task :run => [:require] do
    testcases        = ENV['TESTCASES']
    browsers_to_test = ENV['BROWSERS'] && ENV['BROWSERS'].split(',')
    tests_to_run     = ENV['TESTS'] && ENV['TESTS'].split(',')
    runner           = UnittestJS::WEBrickRunner::Runner.new(:test_dir => SCRIPTY2_TMP_DIR)

    Dir[File.join(SCRIPTY2_TMP_DIR, '*_test.html')].each do |file|
      file = File.basename(file)
      test = file.sub('_test.html', '')
      unless tests_to_run && !tests_to_run.include?(test)
        runner.add_test(file, testcases)
      end
    end
    
    UnittestJS::Browser::SUPPORTED.each do |browser|
      unless browsers_to_test && !browsers_to_test.include?(browser)
        runner.add_browser(browser.to_sym)
      end
    end
    
    trap('INT') { runner.teardown; exit }
    runner.run
  end
  
  task :build => [:clean, :dist] do
    builder = UnittestJS::Builder::SuiteBuilder.new({
      :input_dir  => SCRIPTY2_TEST_UNIT_DIR,
      :assets_dir => SCRIPTY2_DIST_DIR
    })
    selected_tests = (ENV['TESTS'] || '').split(',')
    builder.collect(*selected_tests)
    builder.render
  end
  
  task :clean => [:require] do
    UnittestJS::Builder.empty_dir!(SCRIPTY2_TMP_DIR)
  end
  
  task :require do
    lib = 'vendor/unittest_js/lib/unittest_js'
    unless File.exists?(lib)
      puts "\nYou'll need UnittestJS to run the tests. Just run:\n\n"
      puts "  $ git submodule init"
      puts "  $ git submodule update"
      puts "\nand you should be all set.\n\n"
    end
    require lib
  end
end