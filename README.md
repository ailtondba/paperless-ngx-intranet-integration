# Paperless-ngx Intranet Integration

## Descrição
Sistema de integração do Paperless-ngx com portal intranet para busca e visualização de documentos digitalizados.

## Arquitetura do Sistema
![Diagrama de Arquitetura](DGE-PROJETO-VISUAL.html)

## Funcionalidades
- 🔍 **Busca Avançada**: Interface para busca de documentos por múltiplos critérios
- 📄 **Visualização**: Preview de documentos PDF diretamente na intranet
- 🔐 **Autenticação**: Integração com sistema de autenticação existente
- 📊 **API REST**: Endpoints para integração com outros sistemas
- 🌐 **Interface Web**: Portal responsivo para acesso aos documentos

## Estrutura do Projeto
```
├── API-de-integracao.md          # Documentação da API
├── DEBIAN-SETUP.md               # Configuração no Debian
├── DEMO.md                       # Demonstração do sistema
├── DGE-PROJETO-COMPLETO.md       # Documentação completa
├── DGE-PROJETO-VISUAL.html       # Diagrama visual da arquitetura
├── GUIA-API.md                   # Guia de uso da API
├── INSTALACAO.md                 # Guia de instalação
├── config/                       # Arquivos de configuração
├── exemplos/                     # Exemplos de implementação
└── documentacao.md               # Documentação adicional
```

## Tecnologias Utilizadas
- **Backend**: Paperless-ngx API
- **Proxy**: Nginx
- **Database**: PostgreSQL
- **Frontend**: HTML5, CSS3, JavaScript
- **Containerização**: Docker

## Instalação Rápida

### Pré-requisitos
- Docker e Docker Compose
- Paperless-ngx instalado e configurado
- Nginx configurado

### Passos
1. Clone o repositório:
```bash
git clone https://github.com/ailtondba/paperless-ngx-intranet-integration.git
cd paperless-ngx-intranet-integration
```

2. Configure as variáveis de ambiente:
```bash
cp config/config.example.js config/config.js
# Edite o arquivo config.js com suas configurações
```

3. Execute o sistema:
```bash
docker-compose up -d
```

## Documentação
- [📖 Instalação Completa](INSTALACAO.md)
- [🔧 Configuração Debian](DEBIAN-SETUP.md)
- [🚀 API de Integração](API-de-integracao.md)
- [📋 Guia da API](GUIA-API.md)
- [🎯 Demonstração](DEMO.md)
- [📊 Projeto Completo](DGE-PROJETO-COMPLETO.md)

## Exemplos de Uso
Veja a pasta `exemplos/` para implementações de referência:
- Busca simples e avançada
- Integração com React
- Testes de API
- Interface para intranet Debian

## Contribuição
1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## Licença
Este projeto está sob a licença MIT. Veja o arquivo `LICENSE` para mais detalhes.

## Suporte
Para suporte e dúvidas, abra uma issue no GitHub ou entre em contato através dos canais oficiais.

---

**Desenvolvido para integração com sistemas de intranet corporativa** 🏢