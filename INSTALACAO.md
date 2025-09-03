# 🚀 Guia de Instalação - Integração Paperless-NGX com Intranet

Este guia fornece instruções passo a passo para integrar o Paperless-NGX com sua intranet corporativa.

## 📋 Pré-requisitos

### Sistema
- ✅ Paperless-NGX rodando e acessível
- ✅ Servidor web para hospedar a intranet (Apache, Nginx, IIS, etc.)
- ✅ Navegadores modernos (Chrome 80+, Firefox 75+, Safari 13+, Edge 80+)

### Conhecimentos
- 🔧 Básico de HTML/CSS/JavaScript
- 🔧 Configuração de servidor web
- 🔧 Conceitos de API REST
- 🔧 Configuração de CORS (opcional)

## 🎯 Opções de Integração

### Opção 1: Integração Simples (Recomendada para iniciantes)
- Interface básica com busca por texto
- Configuração mínima
- Ideal para testes e prototipagem

### Opção 2: Integração Avançada (Recomendada para produção)
- Interface completa com filtros avançados
- Paginação e ordenação
- Estatísticas e metadados
- Configuração personalizada

## 📦 Instalação Passo a Passo

### Passo 1: Preparar o Ambiente

1. **Criar diretório da integração**
   ```bash
   mkdir paperless-intranet
   cd paperless-intranet
   ```

2. **Copiar arquivos de exemplo**
   - Copie os arquivos da pasta `integracao-intranet/exemplos/`
   - Escolha entre `busca-simples.html` ou `busca-avancada.html`

### Passo 2: Configurar o Paperless-NGX

1. **Obter Token da API**
   - Acesse o Paperless-NGX: `http://localhost:8000`
   - Faça login como administrador
   - Vá em **Configurações** → **API Tokens**
   - Clique em **Criar Token**
   - Copie o token gerado

2. **Configurar CORS (se necessário)**
   
   Edite o arquivo `docker-compose.env`:
   ```env
   # Adicione estas linhas
   PAPERLESS_CORS_ALLOWED_HOSTS=http://localhost:3000,http://sua-intranet.com
   PAPERLESS_ALLOWED_HOSTS=localhost,sua-intranet.com
   ```
   
   Reinicie o container:
   ```bash
   docker-compose restart webserver
   ```

### Passo 3: Configurar a Integração

1. **Configuração Simples**
   
   Abra `busca-simples.html` e configure:
   ```javascript
   // Configuração da API
   const PAPERLESS_URL = 'http://localhost:8000';
   const API_TOKEN = 'seu_token_aqui';
   ```

2. **Configuração Avançada**
   
   Para `busca-avancada.html`:
   - Configure a URL e token nos campos da interface
   - Ou use o arquivo `config/config.example.js` como base

### Passo 4: Hospedar na Intranet

#### Opção A: Servidor Apache

1. **Copiar arquivos**
   ```bash
   cp -r paperless-intranet/ /var/www/html/documentos/
   ```

2. **Configurar Virtual Host** (opcional)
   ```apache
   <VirtualHost *:80>
       ServerName documentos.intranet.com
       DocumentRoot /var/www/html/documentos
       
       <Directory /var/www/html/documentos>
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

#### Opção B: Servidor Nginx

1. **Configurar site**
   ```nginx
   server {
       listen 80;
       server_name documentos.intranet.com;
       root /var/www/html/documentos;
       index busca-avancada.html;
       
       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```

#### Opção C: IIS (Windows)

1. **Criar aplicação IIS**
   - Abra o Gerenciador do IIS
   - Clique com botão direito em **Sites**
   - **Adicionar Site**
   - Nome: `Documentos`
   - Caminho físico: `C:\inetpub\wwwroot\documentos`
   - Porta: `80`

### Passo 5: Testar a Integração

1. **Teste básico**
   - Acesse a página da intranet
   - Configure URL e Token (se necessário)
   - Faça uma busca simples
   - Verifique se os resultados aparecem

2. **Teste avançado**
   - Teste filtros por categoria
   - Teste filtros por data
   - Teste paginação
   - Teste download de documentos

## 🔧 Configurações Avançadas

### Personalização da Interface

1. **Cores e tema**
   ```css
   :root {
       --primary-color: #667eea;
       --secondary-color: #764ba2;
       --success-color: #28a745;
   }
   ```

2. **Logo da empresa**
   ```html
   <div class="header">
       <img src="logo-empresa.png" alt="Logo">
       <h1>Busca de Documentos</h1>
   </div>
   ```

### Integração com SSO

1. **OAuth2**
   ```javascript
   // Exemplo de integração OAuth2
   async function authenticateWithSSO() {
       const response = await fetch('/auth/oauth2/token', {
           method: 'POST',
           headers: {
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               grant_type: 'authorization_code',
               code: authCode
           })
       });
       
       const data = await response.json();
       return data.access_token;
   }
   ```

### Monitoramento e Logs

1. **Google Analytics**
   ```html
   <!-- Global site tag (gtag.js) - Google Analytics -->
   <script async src="https://www.googletagmanager.com/gtag/js?id=GA_TRACKING_ID"></script>
   <script>
       window.dataLayer = window.dataLayer || [];
       function gtag(){dataLayer.push(arguments);}
       gtag('js', new Date());
       gtag('config', 'GA_TRACKING_ID');
   </script>
   ```

2. **Logs customizados**
   ```javascript
   function logSearch(query, results) {
       fetch('/api/logs/search', {
           method: 'POST',
           headers: {
               'Content-Type': 'application/json'
           },
           body: JSON.stringify({
               timestamp: new Date().toISOString(),
               query: query,
               results_count: results.length,
               user_agent: navigator.userAgent
           })
       });
   }
   ```

## 🛡️ Segurança

### Configurações Essenciais

1. **HTTPS em produção**
   ```nginx
   server {
       listen 443 ssl;
       ssl_certificate /path/to/certificate.crt;
       ssl_certificate_key /path/to/private.key;
   }
   ```

2. **Rate Limiting**
   ```nginx
   http {
       limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
       
       server {
           location /api/ {
               limit_req zone=api burst=20 nodelay;
           }
       }
   }
   ```

3. **Validação de entrada**
   ```javascript
   function sanitizeInput(input) {
       return input
           .replace(/[<>"']/g, '') // Remove caracteres perigosos
           .substring(0, 500); // Limita tamanho
   }
   ```

### Boas Práticas

- ✅ Use HTTPS em produção
- ✅ Implemente rate limiting
- ✅ Valide todas as entradas do usuário
- ✅ Use tokens com escopo limitado
- ✅ Monitore logs de acesso
- ✅ Mantenha o Paperless-NGX atualizado
- ✅ Faça backup regular dos tokens

## 🚨 Solução de Problemas

### Problemas Comuns

1. **Erro de CORS**
   ```
   Access to fetch at 'http://localhost:8000/api/documents/' 
   from origin 'http://localhost:3000' has been blocked by CORS policy
   ```
   
   **Solução:**
   - Configure `PAPERLESS_CORS_ALLOWED_HOSTS` no docker-compose.env
   - Reinicie o container do Paperless-NGX

2. **Token inválido**
   ```
   HTTP 401: Invalid token
   ```
   
   **Solução:**
   - Verifique se o token está correto
   - Gere um novo token se necessário
   - Verifique se o token não expirou

3. **Documentos não aparecem**
   ```
   0 documentos encontrados
   ```
   
   **Solução:**
   - Verifique se há documentos no Paperless-NGX
   - Teste a busca diretamente na interface do Paperless-NGX
   - Verifique os filtros aplicados

4. **Erro de rede**
   ```
   Failed to fetch
   ```
   
   **Solução:**
   - Verifique se o Paperless-NGX está rodando
   - Teste a URL no navegador
   - Verifique configurações de firewall

### Debug

1. **Console do navegador**
   - Pressione F12
   - Vá na aba Console
   - Procure por erros em vermelho

2. **Network tab**
   - Vá na aba Network
   - Faça uma busca
   - Verifique as requisições HTTP

3. **Logs do Paperless-NGX**
   ```bash
   docker-compose logs webserver
   ```

## 📞 Suporte

### Recursos Úteis

- 📖 [Documentação oficial do Paperless-NGX](https://paperless-ngx.readthedocs.io/)
- 🐛 [Issues no GitHub](https://github.com/paperless-ngx/paperless-ngx/issues)
- 💬 [Comunidade no Discord](https://discord.gg/paperless)
- 📧 [Fórum de discussão](https://github.com/paperless-ngx/paperless-ngx/discussions)

### Checklist de Instalação

- [ ] Paperless-NGX rodando e acessível
- [ ] Token da API gerado
- [ ] CORS configurado (se necessário)
- [ ] Arquivos copiados para servidor web
- [ ] URL e token configurados na interface
- [ ] Teste de busca realizado com sucesso
- [ ] HTTPS configurado (produção)
- [ ] Monitoramento implementado
- [ ] Backup dos tokens realizado

---

**🎉 Parabéns!** Sua integração Paperless-NGX + Intranet está pronta!

Para dúvidas ou sugestões, consulte a documentação ou entre em contato com a equipe de TI.