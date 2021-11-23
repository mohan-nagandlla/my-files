# bashrc 

The below command are use to simple cli for kubernetes to access the resources and to perform the activities 
save the below lines of command to /home/${user}/.bashrc

```
alias kg='kubectl get -n '
alias ke='kubectl edit -n '
alias kd='kubectl describe -n'
alias kdl='kubectl delete -n'
alias kl='kubectl logs -n '
alias kdlf='kubectl delete --force --grace-period=0 -n '
alias kx='kubectl exec -it -n '
alias k='kubectl'
alias kc='kubectl create -n'
alias ka='kubectl apply -n'
alias op='kubectl get prometheus-operator -n'
```
