# Using Typescript

## Installation

curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -

### Node.js
```bash
# apt install nodejs
node --version
```

### `npm` (Node Package Manager)
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
```

### Set versions

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

nvm install node
nvm install 8.10.0
nvm install --lts
nvm install 10.16.3
nvm use 10.16.3
```