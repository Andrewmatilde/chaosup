SHELL := /bin/bash

ifneq (,$(wildcard ./.env))
    include .env
    export
endif

# ssh
link:
	ssh -i ssh/root_key root@${VAGRANT_HOST}

# up
up:
	vagrant up

down:
	vagrant destroy -f

#ansible
ansible:
	ansible-playbook -i ${VAGRANT_HOST}, ansible/playbook.yml -u root --private-key ./ssh/root_key

# Create ssh key pair
create-key:
	ssh-keygen -t ed25519 -f vagrant-dev/ssh/root_key -N ""

.PHONY: up ansible link create-key