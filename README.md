# raspberry-pi-ansible

## Ansible Playbook

### k3s

```
$ ansible-playbook -i hosts --limit=pi_kube_servers -e k3s_version=v1.19.4+k3s2 playbooks/k3s.yml
```
