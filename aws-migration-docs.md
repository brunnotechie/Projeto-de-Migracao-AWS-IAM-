# Documentação do Projeto de Migração On-Premises para AWS IAM
Versão 1.0

## Sumário
1. [Visão Geral](#1-visão-geral)
2. [Pré-requisitos](#2-pré-requisitos)
3. [Preparação do Ambiente](#3-preparação-do-ambiente)
4. [Estrutura do Projeto](#4-estrutura-do-projeto)
5. [Script de Migração: Análise Detalhada](#5-script-de-migração-análise-detalhada)
6. [Processo de Execução](#6-processo-de-execução)
7. [Pós-Migração](#7-pós-migração)
8. [Monitoramento e Verificação](#8-monitoramento-e-verificação)
9. [Troubleshooting](#9-troubleshooting)

## 1. Visão Geral

### 1.1 Objetivo
Migrar mais de 100 usuários do ambiente on-premises para AWS IAM, implementando autenticação MFA e gerenciamento automatizado de usuários.

### 1.2 Escopo
- Migração de usuários
- Implementação de MFA
- Criação de grupos e políticas
- Automatização via AWS CLI
- Configuração de políticas de segurança

## 2. Pré-requisitos

### 2.1 Ferramentas Necessárias
- Git Bash instalado
- AWS CLI v2 configurado
- Conta AWS com permissões administrativas
- Python 3.x (opcional, para scripts auxiliares)

### 2.2 Informações Necessárias
- Lista de usuários em formato CSV
- Políticas de acesso definidas
- Mapeamento de grupos e permissões

## 3. Preparação do Ambiente

### 3.1 Configuração do AWS CLI
```bash
aws configure
# Inserir:
# - AWS Access Key ID
# - AWS Secret Access Key
# - Default region (ex: us-east-1)
# - Output format (json)
```

### 3.2 Estrutura de Arquivos
```
projeto-migracao/
├── scripts/
│   ├── migration.sh
│   ├── verify-mfa.sh
│   └── backup.sh
├── policies/
│   ├── mfa-policy.json
│   └── group-policies.json
├── data/
│   └── users.csv
└── logs/
```

## 4. Estrutura do Projeto

### 4.1 Formato do Arquivo users.csv
```csv
username,email,department
joao.silva,joao.silva@empresa.com,TI
maria.santos,maria.santos@empresa.com,TI
```

### 4.2 Estrutura da Política MFA
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowViewAccountInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetAccountPasswordPolicy",
                "iam:ListVirtualMFADevices"
            ],
            "Resource": "*"
        }
        // ... outras permissões ...
    ]
}
```

## 5. Script de Migração: Análise Detalhada

### 5.1 Configuração Inicial
```bash
AWS_PROFILE="default"
GROUP_NAME="TI-Team"
MFA_POLICY_NAME="Force-MFA-Policy"
```
**Função**: Define variáveis globais utilizadas no script.

### 5.2 Criação do Grupo
```bash
aws iam create-group --group-name $GROUP_NAME
```
**Função**: Cria um grupo IAM para organizar os usuários.

### 5.3 Política MFA
```bash
# Criar política MFA
aws iam create-policy --policy-name $MFA_POLICY_NAME --policy-document file://mfa-policy.json
```
**Função**: Implementa a política que força o uso de MFA.

### 5.4 Função de Criação de Usuários
```bash
create_user() {
    local username=$1
    
    # Criar usuário
    aws iam create-user --user-name $username
    
    # Gerar senha inicial
    password=$(openssl rand -base64 12)
    
    # Criar perfil de login
    aws iam create-login-profile --user-name $username --password $password --password-reset-required
    
    # Adicionar ao grupo
    aws iam add-user-to-group --user-name $username --group-name $GROUP_NAME
    
    echo "Usuário $username criado com senha: $password"
}
```
**Função**: Gerencia a criação completa de cada usuário.

## 6. Processo de Execução

### 6.1 Preparação
1. Clone o repositório do projeto
2. Configure o AWS CLI
3. Prepare o arquivo users.csv

### 6.2 Execução do Script
```bash
# 1. Dar permissão de execução
chmod +x scripts/migration.sh

# 2. Executar script
./scripts/migration.sh
```

### 6.3 Ordem de Execução
1. Criação do grupo
2. Implementação da política MFA
3. Processamento do arquivo CSV
4. Criação dos usuários
5. Configuração de senhas
6. Associação aos grupos

## 7. Pós-Migração

### 7.1 Verificação de Usuários
```bash
# Listar todos os usuários criados
aws iam list-users --output table

# Verificar membros do grupo
aws iam get-group --group-name $GROUP_NAME
```

### 7.2 Distribuição de Credenciais
1. Exportar lista de usuários e senhas
2. Enviar credenciais de forma segura
3. Instruir sobre primeira alteração de senha

## 8. Monitoramento e Verificação

### 8.1 Script de Verificação MFA
```bash
#!/bin/bash
# verify-mfa.sh

aws iam list-users --output json | jq -r '.Users[].UserName' | while read username; do
    mfa_devices=$(aws iam list-mfa-devices --user-name $username --output json | jq '.MFADevices | length')
    if [ "$mfa_devices" -eq 0 ]; then
        echo "ALERTA: Usuário sem MFA: $username"
    fi
done
```

### 8.2 Logs e Auditoria
```bash
# Verificar últimos acessos
aws iam generate-credential-report
aws iam get-credential-report

# Verificar políticas aplicadas
aws iam list-attached-user-policies --user-name USERNAME
```

## 9. Troubleshooting

### 9.1 Problemas Comuns
1. **Erro de Permissão**
   ```bash
   aws iam get-user
   # Verificar se suas credenciais têm permissões adequadas
   ```

2. **Falha na Criação de Usuário**
   ```bash
   # Verificar se o usuário já existe
   aws iam get-user --user-name USERNAME
   
   # Deletar usuário se necessário
   aws iam delete-user --user-name USERNAME
   ```

3. **Erro na Política MFA**
   ```bash
   # Verificar política
   aws iam get-policy --policy-arn POLICY_ARN
   
   # Listar versões da política
   aws iam list-policy-versions --policy-arn POLICY_ARN
   ```

### 9.2 Backup e Recuperação
```bash
# Fazer backup das configurações
aws iam get-account-authorization-details > backup.json

# Restaurar usuário específico
aws iam create-user --user-name USERNAME --cli-input-json file://user-backup.json
```

