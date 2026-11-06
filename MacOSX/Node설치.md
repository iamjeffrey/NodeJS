## OSX 27 Over 

🧰 Prerequisites

- Homebrew — the package manager for macOS


🛠️ Editing .zshrc

nano ~/.zshrc
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && \. "/usr/local/opt/nvm/nvm.sh"
[ -s "/usr/local/opt/nvm/etc/bash_completion" ] && \. "/usr/local/opt/nvm/etc/bash_completion"

source $(brew --prefix nvm)/nvm.sh

```

🧠 What does each line do?
export NVM_DIR="$HOME/.nvm": Sets the directory where NVM will keep its files.
The two [ -s ... ] lines conditionally source the main NVM script and the bash completion if they exist.
source $(brew --prefix nvm)/nvm.sh: Dynamically sources NVM from wherever Homebrew has installed it—future-proofing for Apple Silicon paths.

🔁 Reload the Shell
To apply your changes immediately:
source ~/.zshrc

Test it:
nvm --version


🔍 주요 포인트


📝 최종




