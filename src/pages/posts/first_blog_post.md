---
layout: ../../layouts/post.astro
title: "Terraform Basics"
pubDate: 2024-05-06
description: "Purpose and basic commands of Terraform."
author: "eiki"
excerpt: Infrastructure as code. Like a summary of all the features your infrastructure have (storage type and size, ports, cloud), which can be easily read and removed. 
image:
  src: /src/images/terraform_showcase.png
  alt:
tags: ["terraform", "iac", "data engineering"]
---

> [!tldr]
> Infrastructure as code. Like a summary of all the features your infrastructure have (storage type and size, ports, cloud), which can be easily read and removed. 

- não pode mudar recursos imutáveis, como o tipo da [[Máquina Virtual]].
- não gerencia ou atualiza código da infraestrutura (deploy)
- não gerencia recursos que não foram definidos no arquivo Terraform, como cluster da aws.
Providers: código que permite ao Terraform se comunicar com as plataformas de nuvem (Cloud, AWS, GCP, Azure).

### Versionamento
https://spacelift.io/blog/tfenv

Basicamente usar WSL para instalar o versionador do terraform no Windows. Tentar instalar direto pelo power shell ou Prompt de Comando não funciona.

Isso é útil porque, aparentemente, o Terraform tem várias versões e muda bem rápido.

```terraform
> tfenv install
instala alguma versão

> tfenv use
usa alguma versão instalada

> tfenv list
```

### Comandos
- `fmt`: formata os arquivos
- init: get the providers
- plan: what to do
- apply: do what the tf file say
- remove: remove all the resources

```cmd
> terraform fmt // format the mistakes on terraform files
> terraform init // prepare current directory to work with terraform
> terraform plan // creates an execution plan
> terraform apply -auto-approve // create in aws
> terraform apply -destroy -auto-approve // destroy in aws
> terraform destroy // destroi apenas recursos que vc criou
```

#### Dangers of cloud credential exposure
Com grandes permissões, há grandes responsabilidades. Se não limitar o uso do seu serviço, uma conta vazada pode ser utilizada para várias práticas custosas:

- criar uma [[Máquina Virtual]] e minerar bitcoin
- armazenar arquivos gigantes
- command and control servers para botnets

#### Variables
Repare que variáveis não podem ser usadas em conjunto com planos, apenas no modo especulativo.

1. Crie `variables.tf` (pode ser outro nome, mas assim fica intuitivo)
2. Escreva um bloco do tipo `variable`

```terraform
variable "instance_name" {
	description = "Valor para a tag da instância" 
	type = string
	default = "ExampleAppName"
}
```

3. Para chamar, basta fazer `var.instance_name` em qualquer lugar no arquivo. Nesse caso, iremos colocar na tag da instância.

```terraform
 resource "aws_instance" "app_server" {
   ami           = "ami-08d70e59c07c61a3a"
   instance_type = "t2.micro"

   tags = {
+    Name = var.instance_name
   }
 }
```

E mais interessante, pode substituir o valor default durante o `apply`

```terraform
terraform apply -var "OutroNomeQualquer"
```

#### Output

```
> terraform output
```

Basicamente, se você quiser mostrar alguma informação ao executar esse comando, basta criar o bloco `output` com `description` e `value`. Por exemplo: `aws_instance.app_server.public_ip

aws_instance é o tipo do recurso e app_server é o nome que você deu a ele. 

### Locals
Locals você pode usar para guardar ou computar valores complexos dentro do escopo da sua infra. Uma vez que definiu eles, você não consegue alterar de fora, por exemplo, rodando um plan ou apply. 

Variáveis você pode ter diferente valores dependendo do ambiente que você está deployando, por exemplo: em dev se tem uma infra mais enxuta que em prod. Isso só controla usando variáveis.

```locals.tf
locals {
	ip_filepath = "resources/ip.json"
	commom_tags = {
		Environment = var.environment
		OWner = "Bruno"
	}
}
```

```s3.tf
resource "aws_s3_object" "name" {
	bucket = aws_s3_bucket.this.bucket
	key = "${local.ip_filepath}"
	source = ip_filepath
	etag = filemds5(ip_filepath)
}
```

### First Steps 
---
Para começar, é preciso criar um usuário no seu provedor escolhido. Definir quais permissões ele terá, definir as chaves (Access Key e Secret Access Key).

Depois, basta copiar e colar essas chaves em `~/.aws/credentials`

```
[default] // aqui vai ser o nome do usuário
aws_access_key_id = 20character
aws_secret_access_key = 40character
```

Em seguida, crie o seu arquivo ``providers.tf``

```terraform
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
 
# Configure the AWS Provider
provider "aws" {
  region = "sa-east-1"
  shared_credentials_files = "~/.aws/credentials" # acess key e shared acess key
  profile = "eiki"
}
```

## Links

[Inline-style link](https://www.google.com)

[Inline-style link with title](https://www.google.com "Google's Homepage")

[Reference-style link][arbitrary case-insensitive reference text]

[You can use numbers for reference-style link definitions][1]

Or leave it empty and use the [link text itself].

Some text to show that the reference links can follow later.

[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com

## Images

Images included in _\_posts_ folder are lazy loaded.

Inline-style:
![Figure 1: Terraform diagram](/src/images/terraform_showcase.png "Terraform Diagram")

## Table

| Tables        |      Are      | Cool |
| ------------- | :-----------: | ---: |
| col 3 is      | right-aligned | 1600 |
| col 2 is      |   centered    |   12 |
| zebra stripes |   are neat    |    1 |

| Markdown | Less      | Pretty     |
| -------- | --------- | ---------- |
| _Still_  | `renders` | **nicely** |
| 1        | 2         | 3          |

## Syntax highlight

```ts title="astro.config.mjs" showLineNumbers {1-2,5-6}
import { defineConfig } from "astro/config";
import vercelStatic from "@astrojs/vercel/static";

export default defineConfig({
  output: "static",
  adapter: vercelStatic({
    webAnalytics: {
      enabled: true,
    },
  }),
});
```
