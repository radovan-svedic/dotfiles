require 'rake'

task :default => [:install]

desc "Install Dotfiles"
task :install => :pull do
  Rake::Task['install:packages'].invoke
  Rake::Task['install:pygments'].invoke
  Rake::Task['install:fonts'].invoke
  Rake::Task['setup:gitconfig'].invoke
  Rake::Task['setup:tmprop'].invoke

  puts "\n === [\e[0;37mBootstrap Dotfiles\e[0m] ==="
  file_operation(Dir.glob('git/**/*.symlink'))
  file_operation(Dir.glob('ruby/**/*.symlink'))
  file_operation(Dir.glob('system/**/*.symlink'))
  file_operation(Dir.glob('zsh/**/*.symlink'))
  dir_operation(Dir.glob('bin'))

  # Get the Active Shell and Update Not ZSH
  active_shell = %x(echo $SHELL)
  change_shell unless active_shell.include?("zsh")

  Rake::Task['install:vundle'].invoke
  Rake::Task['vundle:update'].invoke
  Rake::Task['vundle:pkgs'].invoke

  puts "[\e[0;32mSuccess\e[0m] Dotfiles Installed! Please close all open terminals."
end

desc "Pull Latest Dotfiles"
task :pull do
  puts "\n === [\e[0;37mUpdating Dotfiles\e[0m] ==="
  run %{ git pull }
end

namespace :setup do

  desc "Setup Git Config"
  task :gitconfig do
    gitcfg = "#{ENV["HOME"]}/.gitconfig"
    unless File.exists?(gitcfg)
      puts "\n === [\e[0;37mBootstraping #{gitcfg}\e[0m] ==="
      puts "[\e[0;34mConfig \e[0m]  $HOME/.gitconfig"
      printf "[\e[0;34mConfig \e[0m]  Enter Git Author Name: "
      git_author_name = STDIN.gets.chomp
      printf "[\e[0;34mConfig \e[0m]  Enter Git Author Email: "
      git_author_email = STDIN.gets.chomp
      run %{ sed -e "s/GIT_AUTHOR_NAME/#{git_author_name}/g" -e "s/GIT_AUTHOR_EMAIL/#{git_author_email}/g" git/gitconfig.symlink.example > git/gitconfig.symlink }
    else
      puts "\n === [\e[0;33m #{gitcfg} exists\e[0m] ==="
    end
  end

  desc "Setup TM Properties Files"
  task :tmprop do
    tmprop = "#{ENV["HOME"]}/.tm_properties"
    unless File.exists?(tmprop)
      puts "\n === [\e[0;37mBootstraping #{tmprop}\e[0m] ==="
      username = `whoami`
      username.chop!
      run %{ sed -e "s/USER_RUBY/#{username}/g" -e "s/USER_PATH/#{username}/g" system/tm_properties.symlink.example > system/tm_properties.symlink }
    else
      puts "\n === [\e[0;33m #{tmprop} exists\e[0m] ==="
    end
  end

  desc "Setup OSX Defaults"
  task :osx do
    osx_defaults if RUBY_PLATFORM.downcase.include?("darwin")
  end

end

namespace :vundle do
  desc "Update Vundle"
  task :update do
    puts "\n === [\e[0;37mUpdate Vundle\e[0m] ==="
    run %{ cd ~/.vim/bundle/vundle && git pull }
  end

  desc "Install All VIM Bundles"
  task :pkgs do
    puts "\n === [\e[0;37mInstall Vundle Pkgs\e[0m] ==="
    run %{ vim +BundleInstall +qall }
  end
end

namespace :install do
  desc "Install Homebrew"
  task :homebrew do
    install_homebrew if RUBY_PLATFORM.downcase.include?("darwin")
  end

  desc "Install Homebrew Package"
  task :packages => :homebrew do
    install_packages if RUBY_PLATFORM.downcase.include?("darwin")
  end

  desc "Install Pygments Highlighter"
  task :pygments do
    install_pygments
  end

  desc "Install Fonts"
  task :fonts do
    install_fonts if RUBY_PLATFORM.downcase.include?("darwin")
  end

  desc "Install Vundle"
  task :vundle do
    puts "\n === [\e[0;37mBootstrap Vundle\e[0m] ==="
    run %{ mkdir -p ~/.vim/bundle/ }
    target = "~/.vim/bundle/vundle"
    if File.directory?(target)
      run %{ git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle }
    end
  end
end

private
def run(cmd)
  puts "[\e[0;33mRunning\e[0m] #{cmd}"
  `#{cmd}` unless ENV['DEBUG']
end

def install_homebrew
  puts "\n === [\e[0;37mBootstrap Homebrew\e[0m] ==="
  run %{ which brew }
  unless $?.success?
    run %{ ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)" }
  end
end

def install_packages
pkgs = [ "ack", "asciidoc", "fop", "bash-completion", "libyaml", "tmux", "git", "spark", "mobile-shell", "fping", "nmap", "wget", "rbenv", "ruby-build", "vim", "jshon", "openssl", "readline", "macvim" ]

  pkgs.each do |p|
    if system("brew list | grep #{p} > /dev/null")
      puts "[\e[0;36mNotice \e[0m]  Package: #{p} already installed."
    else
      puts "[\e[0;36mNotice \e[0m]  Installing Package: #{p}."
      run %{ brew install #{p} }
    end
  end
end

def install_pygments
  run %{ sudo easy_install pip }
  run %{ sudo pip install pygments }
end

def install_fonts
  puts "\n === [\e[0;37mBootstrap Fonts\e[0m] ==="
  run %{ cp -f $HOME/.dotfiles/fonts/* $HOME/Library/Fonts }
end

def osx_defaults
  puts "\n === [\e[0;37mBootstrap OS X\e[0m] ==="
  run %{ $HOME/.dotfiles/bin/osx-defaults }
end

def file_operation(files)
  files.each do |f|
    file = f.split('/').last
    file = file.split('.').first
    source = "#{ENV["PWD"]}/#{f}"
    target = "#{ENV["HOME"]}/.#{file}"

    if File.exists?(target) || File.symlink?(target)
      puts "[\e[0;31mBackup \e[0m]  #{target}"
      run %{ mv "$HOME/.#{file}" "$HOME/.#{file}.backup" }
    end

    run %{ ln -nfs "#{source}" "#{target}" }
  end
end

def dir_operation(dir)
  dir.each do |d|
    source = "#{ENV["PWD"]}/#{d}"
    target = "#{ENV["HOME"]}/#{d}"

    if File.directory?(target)
      puts "[\e[0;31mBackup \e[0m]  #{target}"
      run %{ mv "$HOME/#{d}" "$HOME/#{d}.backup" }
    end

    run %{ ln -nfs "#{source}" "#{target}" }
  end
end

def change_shell
  puts "\n === [\e[0;37mBootstrap Default Shell\e[0m] ==="
  if system('type -f /usr/local/bin/zsh')
    run %{ chsh -s /usr/local/bin/zsh }
    puts "[\e[0;35mChanged\e[0m]  Active ZSH Shell is: #{`which zsh`}"
  else
    run %{ chsh -s /bin/zsh }
    puts "[\e[0;35mChanged\e[0m]  Active ZSH Shell is: #{`which zsh`}"
  end
end
