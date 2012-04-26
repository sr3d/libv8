require 'bundler/setup'
Bundler::GemHelper.install_tasks

require 'rspec/core/rake_task'
RSpec::Core::RakeTask.new(:spec)

V8_Version = Libv8::VERSION.gsub(/\.\d$/,'')
V8_Source = File.expand_path '../vendor/v8', __FILE__

task :checkout do
  sh "git submodule update --init"
  Dir.chdir(V8_Source) do
    sh "git fetch"
    sh "git checkout #{V8_Version}"
    sh "make dependencies"
  end
end

task :compile do
  Dir.chdir(V8_Source) do
    puts "compiling libv8"
    sh "make native GYP_GENERATORS=make"
  end
end

desc "build a binary gem"
task :binary => :compile do
  gemspec = eval(File.read('libv8.gemspec'))
  gemspec.extensions.clear
  gemspec.platform = Gem::Platform.new(RUBY_PLATFORM)

  # We don't need most things for the binary
  gemspec.files = ['lib/libv8.rb', 'lib/libv8/version.rb']
  # V8
  gemspec.files += Dir['vendor/v8/include/*']
  gemspec.files += Dir['vendor/v8/out/native/*']
  FileUtils.mkdir_p 'pkg'
  FileUtils.mv(Gem::Builder.new(gemspec).build, 'pkg')
end

desc "clean up artifacts of the build"
task :clean do
  sh "rm -rf pkg"
  sh "cd #{V8_Source} && git clean -dxf"
end

task :default => [:checkout, :compile, :spec]
task :build => :checkout