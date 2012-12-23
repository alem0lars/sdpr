require 'pathname'
require 'fileutils'
require 'rake/clean'


ROOT_PTH = Pathname.new Dir.getwd
# { Configuration Variables
SRC_PTH = ROOT_PTH.join 'src'
RESOURCES_PTH = SRC_PTH.join 'resources'
OUT_PTH = ROOT_PTH.join 'out'
TMP_PTH = ROOT_PTH.join 'tmp'
APP_NAME = File.basename ROOT_PTH
# }


# { Utility functions
def subpaths(pth)
  Dir.glob(pth.join('**').to_s)
end
# }

# { Tasks and Rules definition
CLEAN.include(*subpaths(TMP_PTH))
CLOBBER.include(*subpaths(TMP_PTH), *subpaths(OUT_PTH))

task :default => [:clobber, :compile]

task :prepare do
  unless SRC_PTH.directory? && SRC_PTH.join("#{APP_NAME}.tex").file?
    fail "The file #{APP_NAME}.tex doesn't exist"
  end

  [TMP_PTH, OUT_PTH].each do |pth|
    FileUtils.mkdir_p(pth) if not pth.directory?
  end
end

task :compile => [:prepare] do
  xelatex_cmd = "xelatex -shell-escape --enable-write18 -output-directory=\"#{OUT_PTH}\" #{APP_NAME}.tex"
  bibtex_cmd = "BIBINPUTS=\"#{SRC_PTH}\" bibtex #{APP_NAME}"
  resources_files = Dir.glob(RESOURCES_PTH.join('*').to_s)

  puts "\nCompiling"

  Dir.chdir(SRC_PTH) do
    sh(xelatex_cmd) { |ok, res|
      Dir.chdir(OUT_PTH) do
        sh(bibtex_cmd) { |ok, res|
          Dir.chdir(SRC_PTH) do
            sh(xelatex_cmd) { |ok, res|
              sh(xelatex_cmd) {
                dirty_files = Dir.glob(RESOURCES_PTH.join('*').to_s) - resources_files
                if (dirty_files.length > 0)
                  sh "rm #{dirty_files.join(' ')}"
                end
              } if (ok)
            } if (ok)
          end
        } if (ok)
      end
    }
  end

  puts ">> #{APP_NAME} compiling (pid=#{$?.pid}) exited with status: #{$?.exitstatus}"
end

task :view => [:compile, :fast_view]

task :fast_view do
  pth = Pathname.new "#{OUT_PTH.join(APP_NAME)}.pdf"
  if pth.file?
    sh "xdg-open #{pth}"
  else
    fail "Cannot open the file #{pth}. Try to run 'rake compile'."
  end
end

