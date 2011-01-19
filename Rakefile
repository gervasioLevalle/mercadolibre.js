require "open-uri"
require "tmpdir"
require "erb"

BUILD = [
  "vendor/cookie.js",
  "tmp/sroc.js",
  "src/mercadolibre.js",
]

directory "pkg"
directory "tmp"

file "sroc.js" => "tmp" do |t|
  target = File.expand_path("tmp/#{t.name}")

  next if File.exist?(target)

  Dir.mktmpdir do |path|
    Dir.chdir(path) do
      system "curl -L -k -s https://github.com/mercadolibre/sroc/tarball/master -o sroc.tar.gz"
      system "tar xf sroc.tar.gz --strip 1"
      system "rake"
      system "cp pkg/sroc.js #{target}"
    end
  end
end

file "mercadolibre.js" => ["pkg", "sroc.js"] do |t|
  File.open("pkg/#{t.name}", "w") do |file|
    file.write ERB.new(File.read("src/build.erb.js")).result(binding)
  end
end

task :minify do
  system "java -jar vendor/yuicompressor-2.4.2.jar --type js pkg/mercadolibre.js -o pkg/mercadolibre.min.js"
end

task :build => "mercadolibre.js" do
end

task :default => [:build, :minify]

task :test => :build do
  require "cutest"
  Cutest.run(Dir["test/test.rb"])
end

task :release do
  `rm -rf pkg`

  Git.each_tag do |tag|
    version = tag.sub(/^v/, "")

    stamp = Time.at(`git log #{tag} --format='%ct' -1`.to_i)
    stamp = stamp.strftime("%Y%m%d%H%M.%S")

    `rake`
    `mv pkg/mercadolibre.js pkg/mercadolibre-#{version}.js`
    `mv pkg/mercadolibre.min.js pkg/mercadolibre-#{version}.min.js`

    `touch -t #{stamp} pkg/mercadolibre-#{version}.js`
    `touch -t #{stamp} pkg/mercadolibre-#{version}.min.js`
  end
end

class Git
  def self.each_tag
    current_branch = `git branch`[/^\* (.*)$/, 1]

    begin
      tags = `git tag -l`.split("\n").sort.reverse

      tags.each do |tag|
        `git checkout -q #{tag} 2>/dev/null`

        unless $?.success?
          $stderr.puts "Need a clean working copy. Please git-stash away."
          exit 1
        end

        puts(tag)

        yield(tag)
      end
    ensure
      `git checkout -q #{current_branch}`
    end
  end
end
