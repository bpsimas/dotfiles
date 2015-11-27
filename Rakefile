ENV['HOMEBREW_CASK_OPTS'] = "--appdir=/Applications"

LINKED_FILES = filemap(
  'vim'                     => '~/.vim',
  'tmux/tmux.conf'          => '~/.tmux.conf',
  'tmux/tmux-osx.conf'      => '~/.tmux-osx.conf',
  'tmux/tmux-linux.conf'    => '~/.tmux-linux.conf',
  'vim/vimrc'               => '~/.vimrc',
  'zsh/zshrc'               => '~/.zshrc',
  'git/gitignore'           => '~/.gitignore',
  'git/git_template'        => '~/.git_template',
  'pry/pryrc'               => '~/.pryrc',
  'ctags/ctags'             => '~/.ctags'
)

def filemap(map)
  map.inject({}) do |result, (key, value)|
    result[File.expand_path(key)] = File.expand_path(value)
    result
  end.freeze
end

def brew_install(package, *options)
  `brew list #{package}`
  return if $?.success?

  sh "brew install #{package} #{options.join ' '}"
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

def unlink_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.symlink?(symlink_path)
    symlink_points_to_path = File.readlink(symlink_path)
    if symlink_points_to_path == original_path
      # the symlink is installed, so we should uninstall it
      rm_f symlink_path, :verbose => true
      backups = Dir["#{symlink_path}*.bak"]
      case backups.size
      when 0
        # nothing to do
      when 1
        mv backups.first, symlink_path, :verbose => true
      else
        $stderr.puts "found #{backups.size} backups for #{symlink_path}, please restore the one you want."
      end
    else
      $stderr.puts "#{symlink_path} does not point to #{original_path}, skipping."
    end
  else
    $stderr.puts "#{symlink_path} is not a symlink, skipping."
  end
end

namespace :install do
  desc 'Update or Install Brew'
  task :brew do
    step 'Homebrew'
    unless system('which brew > /dev/null || ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"')
      raise "Homebrew must be installed before continuing."
    end
  end

  desc 'Install Homebrew Cask'
  task :brew_cask do
    step 'Homebrew Cask'
    unless system('brew tap | grep phinze/cask > /dev/null') || system('brew tap phinze/homebrew-cask')
      abort "Failed to tap phinze/homebrew-cask in Homebrew."
    end

    brew_install 'brew-cask'
  end

  desc 'Install The Silver Searcher'
  task :the_silver_searcher do
    step 'the_silver_searcher'
    brew_install 'the_silver_searcher'
  end

  desc 'Install iTerm'
  task :iterm do
    step 'iterm2'
    unless app? 'iTerm'
      brew_cask_install 'iterm2'
    end
  end

  desc 'Install universal-ctags'
  task :ctags do
    sh 'brew tap universal-ctags/universal-ctags' #TODO: Remove this hack and generalize into a 'tap' method
    sh 'brew install --HEAD universal-ctags'
  end

  desc 'Install reattach-to-user-namespace'
  task :reattach_to_user_namespace do
    step 'reattach-to-user-namespace'
    brew_install 'reattach-to-user-namespace'
  end

  desc 'Install tmux'
  task :tmux do
    step 'tmux'
    brew_install 'tmux'
  end

  desc 'Install git'
  task :git do
    step 'git'
    brew_install 'git'
  end

  desc 'Install hub'
  task :hub do
    step 'hub'
    brew_install 'hub'
  end

  desc 'Install openssl'
  task :openssl do
    step 'openssl'
    brew_install 'openssl'
  end

  # Use gnu-sed, becase OSX's BSD sed is garbage.
  desc 'Install gnu-sed'
  task :sed do
    step 'gnu-sed'
    brew_install 'gnu-sed'
  end

  # Use Homebrew's vim because OSX's is old and shitty and doesn't work with Fugitive nicely.
  # I don't think Macvim needs to be installed, but if you run into trouble, start with that.
  # See Git logs for the previous setup with Macvim
  desc 'Install vim'
  task :vim do
    step 'vim'
    brew_install 'vim'
  end

  desc 'Install Vundle'
  task :vundle do
    step 'vundle'
    install_github_bundle 'gmarik','vundle'
    sh '~/bin/vim -c "BundleInstall" -c "q" -c "q"'
  end

  desc 'Install pick'
  task :pick do
    step 'pick'
    sh 'brew tap thoughtbot/formulae' #TODO: Remove this hack and generalize into a 'tap' method
    sh 'brew install pick'
  end

  desc 'Install rbenv'
  task :rbenv do
    step 'rbenv'
    sh 'brew install rbenv'
  end

  desc 'Install ruby-build'
  task :rubybuild do
    step 'ruby-build'
    sh 'brew install ruby-build'
  end
end

desc 'Symlink dotfiles (and make backups)'
task :symlink do
  step 'Setting up symlinks (making backups of existing files)...'
  LINKED_FILES.each do |orig, link|
    link_file orig, link
  end
  puts 'done!'
end

desc 'Unsymlink dotfiles (and restore backups)'
task :unsymlink do
  puts 'Unsymlinking and restoring backups...'
  LINKED_FILES.each do |orig, link|
    unlink_file orig, link
  end
  puts 'done!'
end

desc 'Install these config files.'
task :install do
  tools = %w(
    brew
    brew_cask
    the_silver_searcher
    iterm
    ctags
    reattach_to_user_namespace
    tmux
    git
    hub
    openssl
    sed
    vim
    pick
    rbenv
    rubybuild
  )

  tools.each do |tool|
    Rake::Task["install:#{tool}"].invoke
  end
  Rake::Task['symlink'].invoke
  Rake::Task['install:vundle'].invoke # After symlinks completed

  puts "  Installation complete!"
  puts
  puts "  You likely need to configure font/colorscheme stuff in iTerm preferences"
  puts
end

desc 'Uninstall these config files.'
task :uninstall do
  Rake::Task['unsymlink'].invoke

  step 'homebrew'
  puts
  puts 'Manually uninstall homebrew if you wish: https://gist.github.com/mxcl/1173223.'

  step 'iterm2'
  puts
  puts 'Run this to uninstall iTerm:'
  puts
  puts '  rm -rf /Applications/iTerm.app'
end

task :default => :install
