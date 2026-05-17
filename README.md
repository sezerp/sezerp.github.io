# sezerp.github.io
math and programming blog


## Getting started

To run this project locally, you need a Ruby environment in your user space (do not use the system Ruby on macOS!). The recommended way is to use the `rbenv` version manager.

### 1. Ruby environment installation (macOS)

Install the `rbenv` version manager and the `ruby-build` package using Homebrew:
```bash
brew install rbenv ruby-build
```

Configure your shell (Zsh) to automatically load `rbenv`:
```bash
echo 'eval "$(rbenv init - zsh)"' >> ~/.zshrc
source ~/.zshrc
```

Install your chosen Ruby version (e.g., 3.2.2) and set it as the default:
```bash
rbenv install 3.2.2
rbenv global 3.2.2
```
*(Make sure, by typing `ruby -v`, that the system is using the newly installed version, and not e.g., `2.6.x`).*

### 2. Install dependencies and run

While in the main project directory, install `bundler` (a tool for managing gems):
```bash
gem install bundler
```

Then install all required project packages:
```bash
bundle install
```

When the installation is complete, start the local server:
```bash
bundle exec jekyll serve
```

Your site will be available at: **http://localhost:4000** (or http://127.0.0.1:4000). Every file edit (except `_config.yml`) will automatically refresh the site in the browser.