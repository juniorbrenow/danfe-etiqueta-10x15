# 🏷️ DANFE Simplificada - Etiqueta 10x15cm

Sistema web para geração de **DANFE Simplificada** em formato de etiqueta **100mm x 150mm (10x15cm)** com código de barras otimizado para leitura em scanners.

![Versão](https://img.shields.io/badge/version-1.0-blue)
![Python](https://img.shields.io/badge/python-3.7+-green)
![Flask](https://img.shields.io/badge/flask-2.0+-red)
![License](https://img.shields.io/badge/license-MIT-yellow)

## ✨ Funcionalidades

- 📄 **Leitura de XML** da NF-e (extrai todos os dados necessários)
- 🏷️ **Geração de etiqueta** no formato 10x15cm (perfeito para impressoras térmicas)
- 📊 **Código de barras Code 128** com largura otimizada (90% da página)
- 🖨️ **Pré-visualização** e impressão direta no navegador
- 📱 **Interface responsiva** com drag & drop para XML
- 🔍 **Dados completos** da NF-e:
  - Emitente (nome, CNPJ, IE, UF)
  - Destinatário (nome, CPF/CNPJ, IE)
  - Produtos (até 2 itens resumidos)
  - Valores (produtos, frete, desconto, ICMS)
  - Chave de acesso com formatação automática
  - Protocolo de autorização

## 🎯 Layout da Etiqueta

## 🚀 Como usar

### Pré-requisitos

```bash
Python 3.7 ou superior
pip (gerenciador de pacotes Python)

git clone https://github.com/seu-usuario/danfe-etiqueta-10x15.git
cd danfe-etiqueta-10x15




Como gerar a etiqueta
Clique na área de upload ou arraste um arquivo XML da NF-e

Clique no botão "Gerar DANFE"

Visualize a etiqueta gerada

Clique em "Imprimir Etiqueta" para enviar para a impressora

Configuração da Impressora
Para impressoras de etiquetas (Zebra, Argox, etc.):

Configure o papel no tamanho 100mm x 150mm

Defina orientação retrato

Ajuste as margens para 0 (zero)

Escala de impressão: 100%

📁 Estrutura do Projeto
text
danfe-etiqueta-10x15/
├── web_app.py          # Aplicação principal
├── README.md           # Documentação
└── requirements.txt    # Dependências
🛠️ Tecnologias Utilizadas
Backend: Python 3.7+ com Flask

PDF Generation: ReportLab

Código de Barras: Code 128 (EAN-128)

Frontend: HTML5, CSS3, JavaScript puro

XML Parsing: ElementTree

📦 Dependências
txt
Flask==2.3.3
reportlab==4.0.4
🔧 Personalização
Alterar tamanho da etiqueta
No arquivo web_app.py, modifique:

python
ETIQUETA_WIDTH = 100 * mm   # Largura em mm
ETIQUETA_HEIGHT = 150 * mm  # Altura em mm
Ajustar largura do código de barras
python
# Em gerar_codigo_barras()
largura_disponivel = ETIQUETA_WIDTH * 0.9  # 90% da página
Modificar fonte ou cores
As cores e fontes estão definidas nas funções de desenho (ex: desenhar_texto_centralizado)

📱 Compatibilidade
✅ Navegadores modernos (Chrome, Firefox, Edge, Safari)

✅ Impressoras térmicas (Zebra, Argox, Elgin, etc.)

✅ Impressoras comuns (A4 com corte)

✅ Mobile (iOS e Android)

🤝 Contribuindo
Contribuições são bem-vindas! Siga os passos:

Faça um Fork do projeto

Crie uma branch para sua feature (git checkout -b feature/AmazingFeature)

Commit suas mudanças (git commit -m 'Add some AmazingFeature')

Push para a branch (git push origin feature/AmazingFeature)

Abra um Pull Request

📄 Licença
Distribuído sob a licença MIT. Veja LICENSE para mais informações.

📧 Contato
Seu Nome - @seutwitter - email@exemplo.com

Link do Projeto: https://github.com/seu-usuario/danfe-etiqueta-10x15

🙏 Agradecimentos
ReportLab - Geração de PDF

Flask - Framework web

Comunidade brasileira de desenvolvedores NF-e

⚠️ Observações Importantes
Este sistema gera uma DANFE Simplificada para uso interno (logística, expedição)

A DANFE oficial continua sendo o documento fiscal padrão

O código de barras segue o padrão Code 128 (EAN-128)

Teste sempre com uma etiqueta antes de usar em produção

🐛 Problemas Conhecidos
XMLs com estrutura não padrão podem precisar de ajustes

Caracteres especiais nos nomes podem ser truncados

Produtos com mais de 2 itens são resumidos

🔄 Roadmap
Suporte para DANFE completa

Impressão em lote (múltiplas notas)

API REST para integração

Banco de dados para histórico

Templates customizáveis

Suporte a QR Code

Desenvolvido com ❤️ para facilitar a vida de quem trabalha com logística e expedição.

text

## 📄 requirements.txt

```txt
Flask==2.3.3
reportlab==4.0.4
🚀 Comandos para criar e enviar o repositório
bash
# 1. Criar uma nova pasta para o projeto
mkdir danfe-etiqueta-10x15
cd danfe-etiqueta-10x15

# 2. Copiar o arquivo web_app.py para a pasta

# 3. Criar os arquivos
echo "Flask==2.3.3" > requirements.txt
echo "reportlab==4.0.4" >> requirements.txt

# 4. Inicializar o Git
git init

# 5. Adicionar todos os arquivos
git add .

# 6. Commit inicial
git commit -m "Initial commit: DANFE Simplificada 10x15cm com código de barras otimizado"

# 7. Criar repositório no GitHub (via site)

# 8. Conectar ao repositório remoto
git remote add origin https://github.com/SEU_USUARIO/danfe-etiqueta-10x15.git

# 9. Enviar para o GitHub
git branch -M main
git push -u origin main
🎨 Badges para o README (opcional)
markdown
![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)
![Flask](https://img.shields.io/badge/Flask-2.0+-red.svg)
![ReportLab](https://img.shields.io/badge/ReportLab-4.0.4-green.svg)
![License](https://img.shields.io/badge/License-MIT-yellow.svg)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)
Este repositório está pronto para ser publicado! O sistema está funcionando perfeitamente e agora você pode compartilhar com sua equipe ou com a comunidade! 🎉

