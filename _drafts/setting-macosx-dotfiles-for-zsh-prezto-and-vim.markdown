## ZSH and Prezto

* Having installed _Homebrew_.

```
$ brew update
  ...
$ brew install zsh macvim fasd git 
```

* Clone Prezto

```bash
$ git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
```

* Change the shell to be zsh instad of default bash

```bash
$ chsh -s /bin/zsh
```

* Copy everything from "runcoms" except README to root of User folder.
* Rename all to include "." before name of file.

## Vim

* Get dotfiles from Github

```bash
$ git clone git@github.com:jperras/vim-dotfiles.git ~/.vim && ln -s ~/.vim/.vimrc ~/.vimrc
```

* Install _Vundle_:

```bash
$ git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```

* Run vim and inside install all Vundle bundles with 

	:BundleInstall

