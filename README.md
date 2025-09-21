# AWS CloudFormation Project — DIO (Fiama)

## 🎯 Header rápido

```md
![AWS CloudFormation](https://img.shields.io/badge/AWS-CloudFormation-orange?logo=amazon-aws)
![IaC](https://img.shields.io/badge/IaC-CloudFormation-blue)
![YAML](https://img.shields.io/badge/Made%20with-YAML-2ea44f)
![Region](https://img.shields.io/badge/Region-sa--east--1-success)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
![Status](https://img.shields.io/badge/Status-Lab%20completo-brightgreen)
```

Infraestrutura como Código (**IaC**) na AWS usando **CloudFormation**.  
Este lab cria uma **VPC pública**, um **bucket S3 privado** (com versionamento) e, opcionalmente, uma **EC2** com Apache para validação. Inclui passo a passo, parâmetros e checklist de evidências (prints).

---

## 🎯 Objetivos
- Praticar IaC em ambiente real na AWS.
- Documentar o processo técnico de forma clara e reprodutível.
- Versionar e compartilhar no GitHub.

---

## 🏗️ Arquitetura do Lab
- **Stack 01 – Networking**: VPC (10.0.0.0/16), Sub-rede pública (10.0.1.0/24), Internet Gateway, Tabela de Rotas (0.0.0.0/0).
- **Stack 02 – Storage**: bucket **S3 privado**, **bloqueio de acesso público**, **criptografia AES-256** e **versionamento**.
- **Stack 03 – (Opcional) EC2 Web**: Security Group liberando **HTTP (80)** e **EC2 t3.micro** com Apache + página “Hello DIO/Fiama”.

> **Custos**: Networking ≈ 0; S3 (baixo, por uso/armazenamento); EC2 pode gerar custo fora do Free Tier — **suba, faça o print e exclua**.

---

## 🗂️ Estrutura do repositório
```
aws-cloudformation-project/
├─ README.md
├─ templates/
│  ├─ 01-networking.yaml
│  ├─ 02-storage.yaml
│  └─ 03-ec2.yaml        # opcional
└─ images/               # evidências (prints)
```

---

## 📦 Templates
- `templates/01-networking.yaml` → VPC, Sub-rede pública, IGW, Route Table, Rota e Associação.  
  **Outputs**: `VpcId`, `PublicSubnetId` (usados pela Stack 03).
- `templates/02-storage.yaml` → S3 privado com bloqueio público, criptografia, versionamento (`EnableVersioning`), e *Outputs* do bucket.
- `templates/03-ec2.yaml` *(opcional)* → Security Group (**HTTP/80**), EC2 (**t3.micro**), *User Data* instalando Apache e **Outputs** `WebUrl`/`InstancePublicIp`.

---

## 🚀 Deploy (Console AWS em PT-BR)
Região sugerida: **sa-east-1 (São Paulo)**.

### Stack 01 — Networking
1. **CloudFormation → Criar pilha → Com novos recursos (padrão)**  
2. **Carregar um arquivo de modelo** → `templates/01-networking.yaml` → **Avançar**  
3. **Nome da pilha**: `dio-networking-fiama` → parâmetros padrão → **Avançar → Avançar → Criar pilha**  
4. Validar em **VPC**: VPC, Sub-rede, Tabela de Rotas (rota `0.0.0.0/0` → IGW).

### Stack 02 — S3
1. **Criar pilha** → `templates/02-storage.yaml` → **Avançar**  
2. **Nome da pilha**: `dio-storage-fiama`  
   - **BucketName**: *único globalmente* (ex.: `aws-cloudformation-project-fiama-2025`)  
   - **EnableVersioning**: `Yes`  
3. **Avançar → Avançar → Criar pilha**  
4. Validar no **S3**: Overview do bucket, **Propriedades** (Bloqueio Público + Criptografia AES-256) e **Versionamento ativado**.

### Stack 03 — EC2 (Opcional)
1. Pegue na Stack 01 (aba **Saídas/Outputs**): `VpcId` e `PublicSubnetId`  
2. **Criar pilha** → `templates/03-ec2.yaml` → **Avançar**  
3. **Nome**: `dio-ec2-fiama`  
   - **VpcId**: *copiar da Stack 01*  
   - **PublicSubnetId**: *copiar da Stack 01*  
   - **InstanceType**: `t3.micro`  
4. **Avançar → Avançar → Criar pilha**  
5. Ao finalizar, na aba **Saídas (Outputs)**, clique em **`WebUrl`** para abrir a página “Hello DIO/Fiama”.

---

## 📸 Checklist de evidências (prints)

### Stack 01 — Networking
- [ ] `01-networking-04-eventos.png` → **Eventos** com `CREATE_COMPLETE`  
- [ ] `01-networking-05-recursos.png` → **Recursos** listados  
- [ ] `01-networking-06-vpc-subrede-rota.png` → VPC, Sub-rede e rota (console VPC)

### Stack 02 — S3
- [ ] `02-s3-04-eventos.png` → **Eventos** com `CREATE_COMPLETE`  
- [ ] `02-s3-05-recursos.png` → **Recursos** (bucket)  
- [ ] `02-s3-06-bucket-overview.png` → Overview do bucket  
- [ ] `02-s3-07-bucket-properties.png` → Propriedades (Bloqueio + Criptografia)  
- [ ] `02-s3-08-bucket-versioning.png` → Versionamento **ativado**

### Stack 03 — EC2 (opcional)
- [ ] `03-ec2-04-eventos.png` → **Eventos** com `CREATE_COMPLETE`  
- [ ] `03-ec2-05-recursos.png` → Instância + SG  
- [ ] `03-ec2-06-ec2-console.png` → Instância “Em execução” com IPv4 público  
- [ ] `03-ec2-07-weburl-hello.png` → Página “Hello DIO/Fiama” aberta

### Cleanup (limpeza)
- [ ] `05-cleanup-02-delete-eventos.png` → **Eventos** com `DELETE_COMPLETE` (de alguma pilha)

> Se perder o `DELETE_COMPLETE`, mude o filtro de status para **Excluídas** no CloudFormation ou reproduza com uma pilha “dummy”.

---

## 🔧 Troubleshooting rápido
- **Bucket name já existe** → troque por outro nome (único global).  
- **Falha ao apagar stack do S3** → esvazie o bucket antes de excluir.  
- **Página da EC2 não abre** → aguarde 1–2 min; confirme **SG porta 80**, Sub-rede pública, IP público e rota `0.0.0.0/0` para o IGW.  
- **Erro ASCII no Security Group** → não use acentos em `GroupDescription`.  
- **Erros YAML** → use somente espaços (sem tabs) e aspas quando houver `:` em descrições.

---

## 💰 Custos e limpeza
- Use a EC2 apenas para validar e **exclua a pilha** depois.  
- No final, exclua as pilhas na **ordem inversa**: `dio-ec2-fiama` → `dio-storage-fiama` (esvaziando o bucket antes) → `dio-networking-fiama`.

---

## ✨ Badges
- AWS CloudFormation • IaC • YAML • São Paulo (sa‑east‑1) • MIT

---

## 🧾 Licença
Este projeto é licenciado sob os termos da **MIT License**.  
Veja o arquivo [`LICENSE`](LICENSE) para mais detalhes.
