# API de Integração - Portal de Busca para Paperless-ngx 🔍

## 📋 Visão Geral

Este projeto é uma página de intranet, contida em um único arquivo HTML, que funciona como um **portal de busca** para uma instância do Paperless-ngx. A interface permite que usuários façam buscas por documentos, visualizem uma prévia completa e realizem o download, tudo isso sem precisar acessar a interface principal do Paperless.

A solução foi projetada para ser servida por um container **Nginx**, que também atua como um **proxy reverso seguro** para autenticar as requisições de preview de documentos.

## ✨ Funcionalidades

- **🔍 Busca Simples**: Campo de busca para consultar documentos por qualquer termo
- **👁️ Visualização Segura**: Exibe o preview completo e interativo do documento dentro de um modal, sem expor o token da API no navegador e sem exigir que o usuário esteja logado no Paperless
- **⬇️ Download Direto**: Permite baixar o arquivo do documento com um único clique
- **🎨 Interface Limpa**: Design responsivo e com feedback claro sobre o status da busca (carregando, sucesso, erro)

## 🔄 Fluxograma de Funcionamento ("Ergonograma")

Existem dois fluxos principais de comunicação: a **Busca de Documentos** e a **Visualização do Preview**.

### Fluxo 1: Busca de Documentos

```mermaid
sequenceDiagram
    participant Navegador as Usuário (Navegador)
    participant Paperless as API Paperless (Porta 1994)

    Usuário->>Navegador: Digita "cnh" e clica "Pesquisar"
    Navegador->>Paperless: Envia requisição GET para /api/documents/?query=cnh
    Note right of Navegador: Inclui o cabeçalho "Authorization: Token ..."
    Note left of Paperless: CORS permite a requisição vinda da porta 2026
    Paperless-->>Navegador: Retorna a lista de documentos (JSON)
    Navegador->>Usuário: Exibe os resultados na tela
```

### Fluxo 2: Visualização do Preview (com Proxy)

```mermaid
sequenceDiagram
    participant Navegador as Usuário (Navegador com iframe)
    participant Nginx as Proxy Nginx (Porta 2026)
    participant Paperless as API Paperless (Porta 1994)

    Usuário->>Navegador: Clica no botão "Visualizar"
    Navegador->>Nginx: Iframe solicita a URL local /preview?docId=151
    Nginx->>Nginx: Recebe o pedido no "location /preview"
    Note right of Nginx: Anexa o cabeçalho "Authorization: Token ..."
    Nginx->>Paperless: Repassa a requisição (já autenticada) para /api/documents/151/preview/
    Note left of Paperless: X-Frame-Options está desativado, permitindo o iframe
    Paperless-->>Nginx: Retorna a página de preview do documento
    Nginx-->>Navegador: Entrega a página de preview para o iframe
    Navegador->>Usuário: Exibe o preview completo no modal
```

## 🛠️ Tecnologias Utilizadas

- **Frontend**: HTML5, CSS3, JavaScript (Vanilla JS com Fetch API)
- **Servidor Web / Proxy**: Nginx
- **Backend**: API REST do Paperless-ngx
- **Ambiente**: Docker & Docker Compose

## ⚙️ Configuração e Implantação

Para implantar esta solução, os seguintes arquivos precisam ser configurados:

### 1. `exemplo-intranet.html` ou `debian-intranet.html`

- Ajustar as variáveis `apiUrl` e `apiToken` no JavaScript para apontar para o servidor Paperless correto
- Alterar a função `visualizarConteudo` para que o `<iframe>` aponte para o proxy local (ex: `/preview?docId=...`)

```javascript
const API_CONFIG = {
    url: 'http://192.168.1.7:1994',  // URL do seu Paperless
    token: 'seu_token_aqui'          // Token da API
};
```

### 2. `docker-compose.env` (do Paperless)

- Adicionar `PAPERLESS_CORS_ALLOWED_HOSTS` para permitir requisições da URL da intranet
- Adicionar `PAPERLESS_WEBSERVER_ENABLE_X_FRAME_HEADERS=false` para permitir preview em iframe

```env
# Configurações CORS
PAPERLESS_CORS_ALLOWED_HOSTS=http://192.168.1.7:2026,http://localhost:2026
PAPERLESS_ALLOWED_HOSTS=192.168.1.7,localhost
PAPERLESS_CSRF_TRUSTED_ORIGINS=http://192.168.1.7:1994,http://localhost:1994

# Permitir iframe para preview
PAPERLESS_WEBSERVER_ENABLE_X_FRAME_HEADERS=false
```

### 3. `nginx.conf`

- Adicionar um novo bloco `server` para escutar na porta da intranet (ex: 2026)
- Este bloco deve ter duas `location`: uma para servir o arquivo HTML e outra para atuar como proxy do preview

```nginx
server {
    listen 2026;
    server_name localhost;

    # Servir a página HTML
    location / {
        root /usr/share/nginx/html/intranet;
        index debian-intranet.html;
        try_files $uri $uri/ =404;
    }

    # Proxy para preview de documentos
    location /preview {
        proxy_pass http://192.168.1.7:1994/api/documents/$arg_docId/preview/;
        proxy_set_header Authorization "Token b73862ce93948cc3466c24b5169a89cebcbafad7c";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 4. `docker-compose.override.yml` (ou `docker-compose.yml`)

- No serviço do Nginx, expor a nova porta da intranet
- Adicionar um novo volume para mapear a pasta com o arquivo HTML

```yaml
services:
  nginx:
    ports:
      - "2026:2026"  # Nova porta para intranet
    volumes:
      - ./integracao-intranet/exemplos:/usr/share/nginx/html/intranet
```

## 🚀 Como Usar

1. **Configurar o ambiente**: Seguir as configurações acima
2. **Reiniciar os containers**: `docker-compose restart`
3. **Acessar a intranet**: Abrir `http://192.168.1.7:2026` no navegador
4. **Fazer buscas**: Digitar termos no campo de busca
5. **Visualizar documentos**: Clicar em "Visualizar" para ver o preview
6. **Baixar documentos**: Clicar em "Baixar" para download direto

## 📁 Estrutura do Projeto

```
integracao-intranet/
├── exemplos/
│   ├── exemplo-intranet.html      # Versão para localhost:8000
│   ├── debian-intranet.html       # Versão para Debian (192.168.1.7:1994)
│   ├── busca-simples.html         # Exemplo básico de busca
│   ├── busca-avancada.html        # Exemplo com filtros avançados
│   └── teste-api.html             # Página para testar conectividade
├── config/
│   └── config.example.js          # Exemplo de configuração
├── docker-compose.env.debian      # Configurações CORS para Debian
├── DEBIAN-SETUP.md               # Guia específico para Debian
├── GUIA-API.md                   # Documentação da API
├── API-de-integracao.md          # Este arquivo
└── README.md                     # Documentação geral
```

## 🔧 Solução de Problemas

### Erro de CORS
- Verificar se `PAPERLESS_CORS_ALLOWED_HOSTS` está configurado corretamente
- Confirmar se a porta da intranet está incluída nas configurações

### Preview não carrega
- Verificar se `PAPERLESS_WEBSERVER_ENABLE_X_FRAME_HEADERS=false` está definido
- Confirmar se o proxy Nginx está funcionando corretamente

### Token inválido
- Gerar um novo token na interface do Paperless
- Atualizar o token tanto no HTML quanto no nginx.conf

## 🎯 Próximos Passos

- [ ] Implementar cache para melhorar performance
- [ ] Adicionar filtros avançados de busca
- [ ] Criar sistema de favoritos
- [ ] Implementar busca por tags
- [ ] Adicionar suporte a múltiplos idiomas

## 📝 Licença

Este projeto é de código aberto e está disponível sob a licença MIT.

---

**Desenvolvido com ❤️ para integração com Paperless-ngx**