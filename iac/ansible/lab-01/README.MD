# Laboratório Ansible: testar conectividade com VMs Ubuntu na Azure

Este exercício didático cria duas VMs Ubuntu e executa o módulo `ping` do Ansible. A configuração abaixo é um exemplo; não representa um ambiente de produção.

## Pré-requisitos

- Azure CLI autenticada com uma assinatura apropriada.
- Ansible instalado na máquina local que atuará como control node.
- Chave SSH gerada e mantida apenas na máquina local.

> **Segurança e custos:** restrinja SSH no Network Security Group ao seu IP administrativo; não deixe a porta 22 aberta para `0.0.0.0/0`. VMs, discos e IPs públicos podem gerar cobrança. Revise os recursos antes de removê-los.

## 1. Criar uma chave local

```bash
ssh-keygen -t ed25519 -f ~/.ssh/azure-ansible-lab
chmod 600 ~/.ssh/azure-ansible-lab
```

A chave privada `~/.ssh/azure-ansible-lab` permanece no seu control node. **Não copie a chave privada para uma VM nem a adicione ao Git.** A chave pública `.pub` é a que será instalada nas VMs.

## 2. Criar o grupo de recursos e as VMs

```bash
az login
az group create --name mdc-rg --location eastus
```

Exemplo para a primeira VM (repita com um nome diferente para a segunda):

```bash
az vm create \
  --resource-group mdc-rg \
  --name mdc-01 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/azure-ansible-lab.pub
```

Antes de conectar, confira no Network Security Group que a regra de SSH permite apenas a origem administrativa necessária. Não publique a saída da CLI se ela contiver endereços ou IDs da sua assinatura.

Obtenha os IPs públicos localmente, sem copiá-los para este repositório:

```bash
az vm show -d -g mdc-rg -n mdc-01 --query publicIps -o tsv
az vm show -d -g mdc-rg -n mdc-02 --query publicIps -o tsv
```

## 3. Criar um inventário local

Crie `inventory.ini` localmente, substituindo os endereços reservados abaixo pelos IPs retornados na sua assinatura:

```ini
[targets]
mdc-01 ansible_host=203.0.113.10 ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/azure-ansible-lab
mdc-02 ansible_host=198.51.100.20 ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/azure-ansible-lab
```

`203.0.113.10` e `198.51.100.20` pertencem a intervalos reservados para documentação e não são os endereços das VMs. Mantenha o inventário real e os IPs da sua assinatura fora do Git.

## 4. Testar a conectividade

```bash
ansible all -i inventory.ini -m ping
```

Resposta `pong` confirma a conectividade SSH e o funcionamento do módulo Ansible para os hosts listados.

## 5. Remover recursos quando terminar

Revise primeiro o conteúdo do grupo de recursos. O comando seguinte remove o grupo e todos os recursos nele contidos, incluindo VMs e discos:

```bash
az group delete --name mdc-rg --yes
```

Use-o somente se tiver certeza de que nada precisa ser preservado.

## Créditos e escopo

Este é um fork/registro de estudo derivado de material didático upstream. Confirme a licença e mantenha os créditos exigidos pela fonte original antes de redistribuir ou apresentar o material como próprio. As instruções acima foram ajustadas para manter a chave privada no control node e usar endereços de documentação no exemplo.
