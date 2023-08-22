# Quickstart

```
mkdir ~/kodekloud
cd ~/kodekloud
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/apple-silicon

./deploy-virtual-machines.sh
```


### expand disks
```
multipass stop kubemaster
multipass stop kubeworker01
multipass stop kubeworker02
multipass set local.kubemaster.disk=30G
multipass set local.kubeworker01.disk=30G
multipass set local.kubeworker02.disk=30G

multipass start kubemaster
multipass start kubeworker01
multipass start kubeworker02
```

### dotfiles

```
export ZSH_DOTFILES_PREP_GITHUB_USER=bossjones

curl -fsSL https://raw.githubusercontent.com/bossjones/zsh-dotfiles-prep/main/bin/zsh-dotfiles-prereq-installer | bash -s -- --debug

chezmoi init -R --debug -v --apply https://github.com/bossjones/zsh-dotfiles.git

sudo chsh -s $(which zsh) ubuntu
```
