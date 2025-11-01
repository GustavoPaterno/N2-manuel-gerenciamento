# 🧩 N2 - Ambiente DevOps com Forgejo e Node.js

Este projeto implementa um fluxo **DevOps completo** utilizando ferramentas open source, atendendo aos requisitos da N2 de Versionamento, Pipelines e Artefatos.

---

## 📁 Estrutura de Pastas

```
/n2
 ├─ /forgejo        → Contém o ambiente Forgejo (Git + Runner + Registry)
 ├─ /app            → Contém a aplicação Node.js (API Express)
```

---

## ⚙️ 1. Subindo o ambiente Forgejo

> Este passo configura o servidor Forgejo, o runner e o registry Docker local.

📍 Caminho:  
```bash
cd /n2/forgejo
```

🚀 Execute:
```bash
docker compose up -d
```

🔹 Isso irá iniciar os seguintes serviços:
- **Forgejo**: Servidor Git e interface web (porta `3001`)
- **Forgejo Runner**: Responsável por executar pipelines
- **Registry**: Armazena imagens Docker geradas no pipeline (porta `5000`)

Após subir, o Forgejo estará disponível em:  
👉 [http://localhost:3001](http://localhost:3001)

---

## 🧠 2. Aplicação Node.js (API Express)

📍 Caminho:  
```bash
cd /n2/app
```

A aplicação é uma API simples com **Express**, definida em `server.js`.

Exemplo básico:
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.send('API funcionando! 🚀');
});

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

---

## 🚀 3. Executando o pipeline de build

Após o ambiente Forgejo estar ativo e o repositório configurado, você pode **fazer um push** na branch `master` para acionar o pipeline.

Exemplo:
```bash
git add .
git commit -m "Atualizando API"
git push origin master
```

O Forgejo Runner irá:
1. Clonar o repositório.
2. Instalar as dependências (`npm install`).
3. Rodar o servidor (`node server.js`).
4. Encerrar o processo automaticamente após 20 segundos.
5. Gerar e publicar a imagem Docker no registry local.

---

## 🐳 4. Publicação no Registry

A imagem Docker gerada será publicada no registry local configurado no Docker Compose.

Exemplo do pipeline (`.forgejo/workflows/build.yml`):

```yaml
name: Build Express API Docker

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: [docker]

    steps:
      - name: Clonar repositório
        run: |
          git clone --branch master http://host.docker.internal:3001/catolica/backend.git .
          echo "Repositório clonado com sucesso"

      - name: Instalar dependências
        run: npm install

      - name: Build da imagem Docker
        run: |
          IMAGE_NAME="localhost:5000/backend:latest"
          docker build -t $IMAGE_NAME .
          docker push $IMAGE_NAME

      - name: Rodar servidor (teste)
        run: |
          node server.js &
          sleep 20
          pkill node
```

✅ Após a execução, a imagem estará disponível no **registry local**:  
[http://localhost:5000/v2/_catalog](http://localhost:5000/v2/_catalog)

---

## 📦 5. Artefatos e Pacotes

Há um pacote npm local (`package1.tgz`) dentro da pasta `packages/` que pode ser instalado com:

```bash
npm install ./packages/package1.tgz
```

---

## 🧾 Resumo rápido

| Diretório | Função | Como executar |
|------------|--------|----------------|
| `/forgejo` | Servidor Git + Runner + Registry | `docker compose up -d` |
| `/app` | API Node.js e pipeline de build | `git push origin master` |

---

## 💡 Dicas
- Certifique-se de que o Docker e o Forgejo estejam em execução antes de dar o push.  
- Se quiser testar o servidor manualmente:  
  ```bash
  cd /n2/app
  node server.js
  ```

---

## 📸 Provas de Execução

Abaixo estão capturas de tela que comprovam o funcionamento do ambiente Forgejo, pipeline e publicação da imagem Docker no registry local.

<p align="center">
  <img src="https://i.imgur.com/w0DjM0I.png" alt="Forgejo rodando" width="600"><br>
  <em>🏗️ Ambiente Forgejo e Runner em execução</em>
</p>

<p align="center">
  <img src="https://i.imgur.com/Wz0CSgF.png" alt="Pipeline executado" width="600"><br>
  <em>⚙️ Execução do pipeline com build automatizado via Forgejo Runner</em>
</p>

<p align="center">
  <img src="https://i.imgur.com/4gcvq5B.png" alt="Imagem publicada no registry" width="600"><br>
  <em>🐳 Imagem Docker publicada no registry local</em>
</p>

<p align="center">
  <img src="https://i.imgur.com/l3ZsAjr.png" alt="Pacote npm local" width="600"><br>
  <em>📦 Instalação e execução do pacote <code>package1.tgz</code></em>
</p>

---


