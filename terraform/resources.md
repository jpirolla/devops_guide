
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

Exemplo:

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

O Terraform permite criar múltiplos recursos idênticos usando o parâmetro `count`. Isso evita a repetição manual de vários blocos semelhantes. Ele espera sempre um inteiro, por exemplo o `count = 4`, e repete o bloco conforme esse inteiro. 
Note também que no código a seguir, o `count.index` indica a iteração atual do loop. 

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


Exemplo real de bloco `resource` com `local_file`


```hcl
variable "arq" {
  default = {
    arq1 = "primeiro arquivo",
    arq2 = "segundo arquivo"
  }
}

resource "local_file" "count_map" {
  count = length(keys(var.arq)) #keys restorna uma lista de chaves do dicionário `arq` e length conta quantos elementos há nessa lista
  
  filename = keys(var.arq)[count.index] #pego o elemento eferente ao meu count 
  content  = var.arq[keys(var.arq)[count.index] 
}
```

Neste caso:

* O **provider** é `local`.
* O **resource type** é `file`.
* O arquivo `arquivo.txt` será criado com o conteúdo especificado.


### Loop com `for each`

```hcl
resource "local_file" "for_each"{ 
  for_each = var.teste
 
  filename = each.key
  content = each.value
} 
```

Nesse caso, não teremos index 0, 1 etc, e sim uma string -> garantimos que o terraform não se perde na ordenação, melhorando o tfstate. 


#### Resumo de resources

* O `resource` é um dos blocos mais importantes do Terraform, pois define o que será provisionado.
* Ele depende de **providers**, que são plugins responsáveis por expor os tipos de recursos.
* O Terraform usa um arquivo de **estado (`tfstate`)** para acompanhar o que foi criado e o que precisa ser alterado.
* Para criar múltiplos recursos, é possível usar o parâmetro `count` para não repetir manualmente cada bloquinho de resource
* O [Terraform Registry](https://registry.terraform.io) é a base que consultamos para entender os parâmetros disponíveis e exemplos de uso de casa provider.



