
## Revisão rápida
### Comandos essenciais para trabalhar com recursos

* `terraform init`: inicializa o projeto e baixa os providers necessários.
* `terraform plan`: mostra as ações que o Terraform irá tomar com base nas mudanças no código.
* `terraform apply`: aplica as mudanças na infraestrutura.
* `terraform destroy`: remove os recursos criados.

---
## Variáveis no Terraform

Variáveis no Terraform são utilizadas para tornar os arquivos de configuração mais flexíveis e reutilizáveis, permitindo separar dados sensíveis ou variáveis de ambiente das definições dos recursos.

### Definindo uma Variável

A definição de uma variável em Terraform é feita com o bloco `variable`, que pode incluir os atributos:

```hcl
variable "nome_da_variavel" {
  type        = tipo
  default     = valor_padrao
  description = "Descrição da variável"
}
```

* **`type`**: Define o tipo da variável. Pode ser `string`, `number`, `bool`, `list`, `map`, entre outros.
* **`default`**: Valor padrão da variável. Se omitido, o Terraform exigirá que um valor seja fornecido externamente.
* **`description`**: Campo opcional, útil para descrever o propósito da variável.

---

### Tipos de Variáveis

#### String

Representa uma cadeia de caracteres.

```hcl
variable "regiao" {
  type        = string
  default     = "us-east-1"
  description = "Região onde os recursos serão provisionados"
}
```

Note que quando passo um default, isso faz com que o tf interprete esse bloco como opcional, ja que temos o valor pré definido. Quando não há o default, o usuário necessariamente precisa passar o campo 

#### Number

Representa valores numéricos inteiros ou decimais.

```hcl
variable "quantidade_instancias" {
  type        = number
  description = "Quantidade de instâncias a serem criadas"
}
```

#### Boolean

Representa valores lógicos: `true` ou `false`.

```hcl
variable "habilitar_monitoramento" {
  type        = bool
  default     = false 
  description = "Define se o monitoramento estará habilitado"
}
```

---

### Acessar os valores das variáveis

Dentro dos arquivos .tf, para acessar o valor de uma variável, utilizamos:
```hcl
var.nome_da_variavel
```
Por exemplo:

```hcl
output "idade" {
  value       = var.idade
  description = "Sua idade"
}
```


#### OBS: Outputs

No Terraform, o bloco `output` é utilizado para exportar valores definidos em um módulo e serão consumidos em outros módulos.


**Resumindo para que serve o `output`**

* Exibir valores no terminal após `terraform apply`;
* Compartilhar dados entre módulos;
* Depurar e verificar valores usados durante a execução;
* Exportar informações para uso externo.


Exemplo

```hcl
variable "idade" {
  type        = number
  description = "Sua idade"
}

output "idade" {
  value       = var.idade
  description = "Sua idade"
}
```

Neste exemplo:

* A variável `idade` é declarada e pode ser passada manualmente ou via outras formas;
* O bloco `output` irá **mostrar o valor da variável `idade` no final da execução** do `terraform apply`.

### Saída esperada no terminal

```text
Outputs:

idade = 21
```



### Tipos Compostos

#### Lista (List)

Uma lista é uma coleção ordenada de valores do mesmo tipo.

```hcl
variable "zonas_disponibilidade" {
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b", "us-east-1c"]
  description = "Zonas de disponibilidade para distribuição das instâncias"
}
```

Para acessar um item da lista:

```hcl
output "primeira_zona" {
  value = var.zonas_disponibilidade[0]
}
```

#### Mapa (Map)

Um mapa é uma coleção de pares chave-valor, semelhante a um dicionário.

```hcl
variable "ambientes" {
  type = map(string)
  default = {
    "dev"  = "10.0.0.0/16"
    "prod" = "172.16.0.0/16"
  }
  description = "CIDRs para os ambientes de desenvolvimento e produção"
}
```

Acesso a um valor do mapa:

```hcl
output "cidr_producao" {
  value = var.ambientes["prod"]
}
```

#### Objeto (Object)

Um objeto permite agrupar diferentes atributos com tipos distintos.

```hcl
variable "usuario" {
  type = object({
    nome      = string
    idade     = number
    ativo     = bool
  })
  default = {
    nome  = "João"
    idade = 30
    ativo = true
  }
}
```

Acesso aos atributos:

```hcl
output "nome_usuario" {
  value = var.usuario.nome
}
```

---

### Variáveis Sensíveis

Quando uma variável é marcada como sensível, seu valor será ocultado no terminal e nos arquivos de estado (`state`), evitando a exposição de informações confidenciais.

```hcl
variable "senha_banco" {
  type      = string
  sensitive = true
  description = "Senha do banco de dados"
}
```

---

### Formas de Atribuir Valores às Variáveis

1. **Entrada interativa via terminal**
   Se a variável não possuir valor padrão, o Terraform solicitará o valor durante a execução do `terraform apply`.

2. **Linha de comando**

```bash
terraform apply -var="quantidade_instancias=3" -var="habilitar_monitoramento=true"
```

3. **Variáveis de ambiente**

Podemos definir variáveis como variáveis de ambiente prefixadas com `TF_VAR_`:

```bash
export TF_VAR_quantidade_instancias=3
terraform apply
```

4. **Arquivos `.tfvars`**

Podemos também criar um `.tfvars`, por exemplo que contenha:

```hcl
quantidade_instancias = 3
habilitar_monitoramento = true
```

Durante a execução, o Terraform carrega automaticamente esse arquivo. Também é possível informar explicitamente com o parâmetro:

```bash
terraform apply -var-file="dev.tfvars"
```
É interessante utilizar o `.tfvars` para gerenciar diferentes configurações por ambiente (desenvolvimento, produção, testes) etc.

---


## Resources


No Terraform, o bloco `resource` é o principal mecanismo utilizado para criar e gerenciar componentes da infraestrutura, já que  através dele é possível definir de forma declarativa tudo o que será provisionado, atualizado ou destruído, em um processo que pode ser entendido como um ciclo **CRUD**:

- **Create**: criar recursos (por exemplo, uma instância EC2 ou um arquivo local)
- **Read**: verificar o estado atual do recurso
- **Update**: modificar propriedades de um recurso existente
- **Delete**: remover o recurso da infraestrutura

---

## Providers e Resources

Os recursos no Terraform estão contidos dentro de **providers**. Um provider é como um **plugin** que implementa tipos de recursos e permite que o Terraform interaja com APIs externas, como as da AWS, Azure, Google Cloud, etc.


Há também alguns providers locais, como o Local, nos quais podemos implementar operações no sistema de arquivos ou ações sem depender de serviços em nuvem.

---

## Estrutura de um resource

A definição de um recurso segue a estrutura:

```hcl
resource "<PROVIDER>_<TIPO_DO_RECURSO>" "<NOME_INTERNO>" {
  # Argumentos definidos na documentação oficial
}
```

Exemplo genérico:

```hcl
resource "aws_s3_bucket" "meu_bucket" {
  bucket = "nome-do-bucket"
  acl    = "private"
}
```

Obs: é importante consultar a [documentação oficial no Terraform Registry](https://registry.terraform.io/) para conhecer os **inputs exigidos** por cada tipo de recurso. Cada recurso possui seus próprios parâmetros.

---

## Como o Terraform detecta mudanças?

O Terraform utiliza um arquivo chamado `terraform.tfstate` para armazenar o **estado da infraestrutura**. Esse arquivo contém metadados sobre todos os recursos gerenciados.

Quando um comando como `terraform plan` é executado, o Terraform compara o que está definido no código com o que está registrado no `tfstate`. Essa comparação é chamada de **diff de estado**, e permite que o Terraform aplique apenas as mudanças necessárias.

---

## Repetição de recursos com `count`

O Terraform permite criar múltiplos recursos idênticos usando o parâmetro `count`. Isso evita a repetição manual de vários blocos semelhantes.

### Exemplo com `count`:

```hcl
resource "local_file" "count" {
  count    = 4
  filename = "./arquivo_${count.index}"
  content  = "Esse é o arquivo ${count.index}"
}
```

Este código cria quatro arquivos:

* `arquivo_0`
* `arquivo_1`
* `arquivo_2`
* `arquivo_3`

### Equivalente manual (sem `count`):

```hcl
resource "local_file" "arquivo0" {
  filename = "./arquivo_0"
  content  = "Esse é o arquivo 0"
}

resource "local_file" "arquivo1" {
  filename = "./arquivo_1"
  content  = "Esse é o arquivo 1"
}

# ... e assim por diante
```

## Exemplo real de bloco `resource` com `local_file`

```hcl
resource "local_file" "arquivo_exemplo" {
  filename = "arquivo.txt"
  content  = "Conteúdo gerado com Terraform."
}
```

Neste caso:

* O **provider** é `local`.
* O **resource type** é `file`.
* O arquivo `arquivo.txt` será criado com o conteúdo especificado.


#### Resumo de resources

* O `resource` é um dos blocos mais importantes do Terraform, pois define o que será provisionado.
* Ele depende de **providers**, que são plugins responsáveis por expor os tipos de recursos.
* O Terraform usa um arquivo de **estado (`tfstate`)** para acompanhar o que foi criado e o que precisa ser alterado.
* Para criar múltiplos recursos, é possível usar o parâmetro `count` para não repetir manualmente cada bloquinho de resource
* O [Terraform Registry](https://registry.terraform.io) é a base que consultamos para entender os parâmetros disponíveis e exemplos de uso de casa provider.



