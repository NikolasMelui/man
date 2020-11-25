### Main:

- Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

- go

### Homebrew:

- ansible, autojump, curl, htop, nmap, ranger, telnet, thefuck, tig, tmux, tree, vim, wget, zsh

```bash
  brew install ansible autojump curl htop nmap ranger telnet thefuck tig tmux tree vim wget zsh
```

- nerdfonts

```bash
brew tap homebrew/cask-fonts
brew cask install font-hack-nerd-font)
```

- postgres

```bash
brew install postgresql
brew services start postgresql
```

### Node.js:

- Check the node.js version:

```bash
node -v
```

- Remove all about an old node.js:
```bash
brew uninstall node
sudo rm -rf /usr/local/{lib/node{,/.npm,yarn,_modules},bin,share/man}/{npm*,node*,yarn*,man1/node*}
sudo rm -rf /opt/local/bin/node /opt/local/include/node /opt/local/lib/node_modules
sudo rm -rf /usr/local/bin/npm /usr/local/bin/yarn /usr/local/share/man/man1/node.1 /usr/local/lib/dtrace/node.d
```

- Install the nvm (without using the homebrew):

```bash
https://github.com/nvm-sh/nvm#installing-and-updating
```

- Check the nvm version:

```bash
nvm --version
```

- Install the necessary node.js version:

``` bash
nvm install --lts
```

- Set the node.js version as the default:

``` bash
nvm use default
```

- Check the .npmrc file:

``` bash
ls ~ | grep .npmrc
```

- Install yarn using npm:

``` bash
npm i -g yarn
```

- Check for all node.js and npm versions:

``` bash
where node
which node
where npm
where npm
```

- Update the npm:

``` bash
npm i -g npm
```

- Check the current version of node.js:

``` bash
nvm current
```

- Check the node.js was installed by the current path:

```bash
/Users/nikolasmelui/.nvm/versions/node/${version}/bin/npm
```

### Desktops:

- Google Chrome

- Firefox

- IntellijIDEA

- VSCode

- Postman

- Wireshark

- Iterm2

- Docker

- TunnelBlick

- Karabiner Elements

- Spectacle

- Slack

- Telegram
