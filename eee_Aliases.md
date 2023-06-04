### Aliases 
alias kgp="kubectl get pods -A | grep -V Running"
alias kg="kubectl get"
alias kcc="kubectl config current-context"
alias kcg="kubectl config get-contexts"
alias kcu="kubectl config use-context"

To make aliases permanant.

### Update ~/.bash_aliases
vi ~/.bash_aliases


alias k='kubectl'
alias kg='kubectl get '

Reload 
source ~/.bash_aliases

