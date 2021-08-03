# Bash Exam Config

## VIM YAML

```bash
sudo apt install bash-completion
```

```bash
cat << EOF >> ~/.vimrc
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 expandtab
autocmd FileType py setlocal ts=4 sts=4 sw=4 expandtab
set paste
colorscheme default
let g:indentLine_char = '⦙'
EOF
```

## Alias Bash

```bash
cat << EOF >> .bashrc
# Kubernetes
alias k=kubectl
alias kdp='kubectl delete --force --grace-period=0'
alias kr='kubectl run'

export do="-o yaml --dry-run=client"
source <(kubectl completion bash)
complete -F __start_kubectl k
set -o vi
EOF
```

## Set default namespace

```
kubectl config set-context --current --namespace=my-namespace
```
