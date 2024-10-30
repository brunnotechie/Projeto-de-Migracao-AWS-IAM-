# Guia de Utilização do AWS CLI com Git Bash

## 1. Instalação dos Componentes

### Git Bash
1. Baixe o Git para Windows em: https://gitforwindows.org/
2. Durante a instalação, mantenha as opções padrão
3. Certifique-se de selecionar "Use Git and optional Unix tools from the Command Prompt"

### AWS CLI
1. Baixe o instalador AWS CLI v2 para Windows: https://awscli.amazonaws.com/AWSCLIV2.msi
2. Execute o instalador
3. Abra o Git Bash e verifique a instalação:
```bash
aws --version
```

## 2. Configuração Inicial

### Configurar Credenciais AWS
```bash
aws configure
```
Você precisará fornecer:
- AWS Access Key ID
- AWS Secret Access Key
- Default region name (ex: us-east-1)
- Default output format (json)

### Verificar Configuração
```bash
aws configure list
aws sts get-caller-identity
```

## 3. Comandos Essenciais

### Gerenciamento de Usuários IAM
```bash
# Listar usuários
aws iam list-users

# Criar novo usuário
aws iam create-user --user-name USERNAME

# Adicionar usuário a um grupo
aws iam add-user-to-group --user-name USERNAME --group-name GROUPNAME

# Criar acesso programático
aws iam create-access-key --user-name USERNAME
```

### Gerenciamento de Grupos
```bash
# Listar grupos
aws iam list-groups

# Criar grupo
aws iam create-group --group-name GROUPNAME

# Listar usuários em um grupo
aws iam get-group --group-name GROUPNAME
```

### Gerenciamento de MFA
```bash
# Listar dispositivos MFA
aws iam list-virtual-mfa-devices

# Criar dispositivo MFA virtual
aws iam create-virtual-mfa-device --virtual-mfa-device-name USERNAME_MFA --outfile QRCode.png --bootstrap-method QRCodePNG

# Habilitar MFA para usuário
aws iam enable-mfa-device --user-name USERNAME --serial-number arn:aws:iam::ACCOUNT-ID:mfa/USERNAME_MFA --authentication-code-1 CODE1 --authentication-code-2 CODE2
```

## 4. Scripts Úteis

### Verificar Status MFA dos Usuários
```bash
#!/bin/bash

echo "Verificando usuários sem MFA..."
aws iam list-users --output json | jq -r '.Users[].UserName' | while read username; do
    mfa_devices=$(aws iam list-mfa-devices --user-name $username --output json | jq '.MFADevices | length')
    if [ "$mfa_devices" -eq 0 ]; then
        echo "Usuário sem MFA: $username"
    fi
done
```

### Backup de Configurações IAM
```bash
#!/bin/bash

BACKUP_DIR="aws_backup_$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup de usuários
aws iam list-users --output json > "$BACKUP_DIR/users.json"

# Backup de grupos
aws iam list-groups --output json > "$BACKUP_DIR/groups.json"

# Backup de políticas
aws iam list-policies --scope Local --output json > "$BACKUP_DIR/policies.json"

echo "Backup completo em: $BACKUP_DIR"
```

## 5. Boas Práticas

1. **Aliases Úteis**
Adicione ao seu ~/.bashrc:
```bash
alias awsls='aws iam list-users --output table'
alias awswho='aws sts get-caller-identity'
alias awsgroup='aws iam list-groups --output table'
```

2. **Perfis Múltiplos**
Configure diferentes perfis no ~/.aws/credentials:
```ini
[default]
aws_access_key_id=YOUR_ACCESS_KEY
aws_secret_access_key=YOUR_SECRET_KEY

[prod]
aws_access_key_id=PROD_ACCESS_KEY
aws_secret_access_key=PROD_SECRET_KEY
```

Use com:
```bash
aws s3 ls --profile prod
```

## 6. Solução de Problemas Comuns

1. **Erro de Credenciais**
```bash
aws configure list
aws configure set aws_access_key_id NEW_KEY
aws configure set aws_secret_access_key NEW_SECRET
```

2. **Erro de Região**
```bash
aws configure set region us-east-1
```

3. **Limpar Credenciais**
```bash
rm ~/.aws/credentials
rm ~/.aws/config
```

## 7. Dicas de Segurança

1. Sempre use MFA para contas com acesso administrativo
2. Evite armazenar credenciais em scripts
3. Use roles temporárias quando possível
4. Faça rotação regular das access keys
5. Monitore os logs do CloudTrail

