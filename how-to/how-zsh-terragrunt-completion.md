# How to Add Terragrunt Autocompletion to Zsh

`Terragrunt` is an infrastructure as code (IaC) tool for `Terraform`.
It allows you to use Terraform to deploy infrastructure across multiple cloud providers.

Zsh autocompletion makes it quick and easy to enter Terragrunt commands. 


To add Terragrunt autocompletion to Zsh, follow these steps:
-  Add the following code to the $HOME/.zshrc file and save file:
```
function terragrunt-completition() {
	complete -o nospace -C /opt/homebrew/bin/terragrunt -C /opt/homebrew/bin/terraform terragrunt
}

```
- Open terminal/console and load function `terragrunt-completition`
```
$ terragrunt-completition
```

- Test completition using `first chars` and pressint `tab`
```
$ terragrunt state <Press TAB>
list              mv                pull              push              replace-provider  rm                show       

```
