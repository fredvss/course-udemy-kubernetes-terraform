# Terraform Workflow

## Visão geral

Terraform é uma ferramenta de infraestrutura como código que permite criar recursos na AWS de forma declarativa.

---

## Fluxo de execução

```
write -> plan -> apply
```

---

## Etapas detalhadas

### 1. Escrever código

Arquivos `.tf` definem recursos:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

---

### 2. Inicializar

```bash
terraform init -backend-config=backend.hcl
```

Baixa providers e configura backend. O arquivo `backend.hcl` contém dados sensíveis (nome do bucket com account ID) e **não é versionado**. Copie `backend.hcl.example` para `backend.hcl` e preencha com os valores reais:

```bash
cp backend.hcl.example backend.hcl
```

---

### 3. Planejar

```bash
terraform plan
```

Mostra alterações que serão feitas.

---

### 4. Aplicar

```bash
terraform apply
```

Cria recursos na AWS.

---

## Terraform State

Arquivo `terraform.tfstate` guarda o estado atual da infraestrutura.

---

## Backend remoto (S3)

### Configurar bucket

Criar o bucket no S3 e verificar:

```bash
aws s3 ls
aws s3 ls s3://[bucket_name]
```

Configurar o Terraform para usar o S3 como backend seguindo a [documentação](https://developer.hashicorp.com/terraform/language/backend/s3) e alterando o arquivo `provider.tf`.

### Partial backend configuration

O bloco `backend` no `provider.tf` não aceita variáveis — os valores precisam ser literais. Para não expor dados sensíveis no repositório (como o account ID no nome do bucket), usamos **partial backend configuration**:

- `provider.tf` define apenas `key` e `region` (sem `bucket`)
- `backend.hcl` contém o `bucket` com o valor real → **ignorado pelo git**
- `backend.hcl.example` contém o placeholder → **versionado como referência**

```bash
# Primeira vez: copie o exemplo e preencha com seus valores
cp backend.hcl.example backend.hcl

# Inicializar passando o arquivo de backend
terraform init -backend-config=backend.hcl
```

---

## Passo a passo completo

### 1. Adicionar providers

Os providers podem ser acessados em [terraform providers](https://registry.terraform.io/browse/providers). Geralmente é criado o arquivo `provider.tf` com as configs do provider.

```bash
terraform init -backend-config=backend.hcl
```

### 2. Criação de recursos

A criação dos recursos segue a [documentação de recursos](https://registry.terraform.io/providers/hashicorp/aws/latest/docs). Geralmente são criados arquivos para cada tipo de recurso (ex: `vpc.tf`, `subnets.tf`).

```bash
terraform plan
terraform apply
# ou com aprovação automática:
terraform apply -auto-approve
```

### 3. Formatação e validação (linter)

Verificação recursiva dos arquivos:

```bash
terraform fmt -recursive -check
echo $?
```

Execução da formatação:

```bash
terraform fmt -recursive
```

Validação:

```bash
terraform validate
```

> Seria interessante adicionar esses steps em um pre-commit.

### 4. Terraform console

O `terraform console` pode ser usado para testar funções:

```bash
terraform console
cidrsubnet("10.0.0.0/8", 8, 1)
```

### 5. Destruir infraestrutura

```bash
terraform plan -destroy
terraform apply -destroy
# ou:
terraform apply -destroy -auto-approve
```

---

## Comandos úteis (resumo)

| Comando | Descrição |
|---|---|
| `terraform init` | Inicializa providers e backend |
| `terraform plan` | Mostra o plano de execução |
| `terraform apply` | Aplica as mudanças |
| `terraform fmt` | Formata os arquivos |
| `terraform validate` | Valida a sintaxe |
| `terraform console` | Console interativo para testar funções |
| `terraform destroy` | Destroi a infraestrutura |
