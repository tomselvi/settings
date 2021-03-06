ENV['HOMEBREW_CASK_OPTS'] = "--appdir=/Applications"
require 'rbconfig'
def os
  @os ||= (
    host_os = RbConfig::CONFIG['host_os']
    case host_os
    when /mswin|msys|mingw|cygwin|bccwin|wince|emc/
      :windows
    when /darwin|mac os/
      :macosx
    when /linux/
      :linux
    when /solaris|bsd/
      :unix
    else
      raise Exception, "unknown os: #{host_os.inspect}"
    end
  )
end

def brew_install(package, *options)
  if os == :macosx
    `brew list #{package}`
    return if $?.success?

    sh "brew install #{package} #{options.join ' '}"
  elsif os == :linux
    `dpkg -s #{package}`
    return if $?.success?

    sh "sudo apt-get install #{package}"
  else
    raise Exception, "this installer only supports linux or mac os"
  end
end

def install_github_bundle(user, package)
  unless File.exist? File.expand_path("~/.vim/bundle/#{package}")
    sh "git clone https://github.com/#{user}/#{package} ~/.vim/bundle/#{package}"
  end
end

def brew_cask_install(package, *options)
  output = `brew cask info #{package}`
  return unless output.include?('Not installed')

  sh "brew cask install #{package} #{options.join ' '}"
end

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end

def app_path(name)
  path = "/Applications/#{name}.app"
  ["~#{path}", path].each do |full_path|
    return full_path if File.directory?(full_path)
  end

  return nil
end

def app?(name)
  return !app_path(name).nil?
end

def get_backup_path(path)
  number = 1
  backup_path = "#{path}.bak"
  loop do
    if number > 1
      backup_path = "#{backup_path}#{number}"
    end
    if File.exists?(backup_path) || File.symlink?(backup_path)
      number += 1
      next
    end
    break
  end
  backup_path
end

def link_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.exists?(symlink_path) || File.symlink?(symlink_path)
    if File.symlink?(symlink_path)
      symlink_points_to_path = File.readlink(symlink_path)
      return if symlink_points_to_path == original_path
      # Symlinks can't just be moved like regular files. Recreate old one, and
      # remove it.
      ln_s symlink_points_to_path, get_backup_path(symlink_path), :verbose => true
      rm symlink_path
    else
      # Never move user's files without creating backups first
      mv symlink_path, get_backup_path(symlink_path), :verbose => true
    end
  end
  ln_s original_path, symlink_path, :verbose => true
end

namespace :install do
  desc 'Update or Install Brew'
  task :brew do
    step 'Homebrew'
    unless os != :macosx || system('which brew > /dev/null || ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"')
      raise "Homebrew must be installed before continuing."
    end
  end

  desc 'Install Homebrew Cask'
  task :brew_cask do
    step 'Homebrew Cask'
    if os == :macosx
      unless system('brew tap | grep phinze/cask > /dev/null') || system('brew tap phinze/homebrew-cask')
        abort "Failed to tap phinze/homebrew-cask in Homebrew."
      end

      brew_install 'brew-cask'
    end
  end

  desc 'Install The Silver Searcher'
  task :the_silver_searcher do
    step 'the_silver_searcher'
    if os == :macosx
      brew_install 'the_silver_searcher'
    else
      brew_install 'software-properties-common'
      sh 'apt-add-repository ppa:mizuno-as/silversearcher-ag'
      sh 'apt-get update'
      brew_install 'silversearcher-ag'
    end
  end

  desc 'Install iTerm'
  task :iterm do
    step 'iterm2'
    if os == :macosx
      unless app? 'iTerm'
        brew_cask_install 'iterm2'
      end
    end
  end

  desc 'Install ctags'
  task :ctags do
    step 'ctags'
    brew_install 'ctags'
  end

  desc 'Install reattach-to-user-namespace'
  task :reattach_to_user_namespace do
    step 'reattach-to-user-namespace'
    if os == :macosx
      brew_install 'reattach-to-user-namespace'
    end
  end

  desc 'Install tmux'
  task :tmux do
    step 'tmux'
    brew_install 'tmux'
  end

  desc 'Install tig'
  task :tig do
    step 'tig'
    brew_install 'tig'
  end

  desc 'Install zsh'
  task :zsh do
      step 'zsh'
      brew_install 'zsh'
  end

  desc 'Install tig'
  task :tig do
      step 'tig'
      brew_install 'tig'
  end

  desc 'Install cmatrix'
  task :cmatrix do
      step 'cmatrix'
      brew_install 'cmatrix'
  end

  desc 'Install MacVim'
  task :macvim do
    step 'MacVim'
    if os == :macosx
      unless app? 'MacVim'
        brew_cask_install 'macvim'
      end

      bin_vim = File.expand_path('~/bin/vim')
      FileUtils.mkdir_p(File.dirname(bin_vim))
      unless File.executable?(bin_vim)
        File.open(bin_vim, 'w', 0744) do |io|
          io << <<-SHELL
  #!/bin/bash
  exec /Applications/MacVim.app/Contents/MacOS/Vim "$@"
          SHELL
        end
      end
    end
  end

  desc 'Install Vundle'
  task :vundle do
    step 'vundle'
    install_github_bundle 'gmarik','vundle'
    sh 'vim -c "BundleInstall" -c "q" -c "q"'
  end
end

desc 'Install these config files.'
task :default do
  Rake::Task['install:brew'].invoke
  Rake::Task['install:brew_cask'].invoke
  Rake::Task['install:the_silver_searcher'].invoke
  Rake::Task['install:iterm'].invoke
  Rake::Task['install:ctags'].invoke
  Rake::Task['install:reattach_to_user_namespace'].invoke
  Rake::Task['install:tmux'].invoke
  Rake::Task['install:tig'].invoke
  Rake::Task['install:macvim'].invoke

  # TODO install gem ctags?
  # TODO run gem ctags?

  step 'symlink'
  link_file 'vim'                   , '~/.vim'
  if os == :macosx
    link_file 'tmux.conf'           , '~/.tmux.conf'
  else
    link_file 'tmux.linux.conf'     , '~/.tmux.conf'
  end
  link_file 'vimrc'                 , '~/.vimrc'
  link_file 'vimrc.bundles'         , '~/.vimrc.bundles'
  link_file 'zshrc'                 , '~/.zshrc'
  link_file 'rc'                    , '~/.rc'
  unless File.exist?(File.expand_path('~/.vimrc.local'))
    cp File.expand_path('vimrc.local'), File.expand_path('~/.vimrc.local'), :verbose => true
  end
  unless File.exist?(File.expand_path('~/.vimrc.bundles.local'))
    cp File.expand_path('vimrc.bundles.local'), File.expand_path('~/.vimrc.bundles.local'), :verbose => true
  end

  # Install Vundle and bundles
  Rake::Task['install:vundle'].invoke

  if os == :macosx
    step 'iterm2 colorschemes'
    colorschemes = `defaults read com.googlecode.iterm2 'Custom Color Presets'`
    dark  = colorschemes !~ /Solarized Dark/
    light = colorschemes !~ /Solarized Light/
    sh('open', '-a', '/Applications/iTerm.app', File.expand_path('iterm2-colors-solarized/Solarized Dark.itermcolors')) if dark
    sh('open', '-a', '/Applications/iTerm.app', File.expand_path('iterm2-colors-solarized/Solarized Light.itermcolors')) if light
    step 'iterm2 profiles'
  end
  puts
  puts "  Your turn!"
  puts
  puts "  Go and manually set up Solarized Light and Dark profiles in iTerm2."
  puts "  (You can do this in 'Preferences' -> 'Profiles' by adding a new profile,"
  puts "  then clicking the 'Colors' tab, 'Load Presets...' and choosing a Solarized option.)"
  puts "  Also be sure to set Terminal Type to 'xterm-256color' in the 'Terminal' tab."
  puts
  puts "  Enjoy!"
  puts
end
