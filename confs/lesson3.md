kubectl config view --minify | grep namespace: 

to get teh current namespace
kubectl config set-context --current --namespace=monitoring
to switch teh namespace
kubectl config set-context --current --namespace=default
to create a short cut
alias cur="kubectl config view --minify | grep namespace:"