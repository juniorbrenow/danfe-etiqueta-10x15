#  - DANFE Etiqueta 10x15cm 
from flask import Flask, render_template_string, request, send_file, jsonify
import xml.etree.ElementTree as ET
from reportlab.lib.pagesizes import mm
from reportlab.pdfgen import canvas
from reportlab.lib import colors
from reportlab.graphics.barcode import code128
from datetime import datetime
import io
import math

app = Flask(__name__)

# Tamanho da etiqueta: 100mm x 150mm (largura x altura)
ETIQUETA_WIDTH = 100 * mm   # 10cm
ETIQUETA_HEIGHT = 150 * mm  # 15cm

HTML_TEMPLATE = '''
<!DOCTYPE html>
<html>
<head>
    <title>DANFE Simplificada - Etiqueta 10x15</title>
    <meta charset="UTF-8">
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            max-width: 500px;
            margin: auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            overflow: hidden;
        }
        .header {
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            color: white;
            padding: 15px;
            text-align: center;
        }
        .header h1 { font-size: 18px; }
        .header p { font-size: 10px; opacity: 0.9; }
        .content { padding: 20px; }
        .upload-area {
            border: 2px dashed #2a5298;
            border-radius: 10px;
            padding: 30px;
            text-align: center;
            background: #f8f9fa;
            cursor: pointer;
        }
        .upload-area:hover { background: #e8f0fe; }
        .file-input { display: none; }
        .file-label {
            background: #2a5298;
            color: white;
            padding: 8px 20px;
            border-radius: 20px;
            cursor: pointer;
            display: inline-block;
            font-size: 12px;
        }
        .btn-generate {
            background: #27ae60;
            color: white;
            padding: 10px;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            width: 100%;
            margin-top: 15px;
            font-size: 14px;
            font-weight: bold;
        }
        .btn-generate:disabled {
            background: #95a5a6;
            cursor: not-allowed;
        }
        .preview {
            margin-top: 20px;
            display: none;
            text-align: center;
        }
        .preview.show { display: block; }
        .preview-iframe {
            width: 100%;
            height: 450px;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        .btn-imprimir {
            background: #3498db;
            color: white;
            padding: 8px 20px;
            border: none;
            border-radius: 20px;
            cursor: pointer;
            margin-top: 10px;
            font-size: 12px;
        }
        .status {
            margin-top: 15px;
            padding: 10px;
            border-radius: 10px;
            text-align: center;
            font-size: 12px;
            display: none;
        }
        .status.success {
            display: block;
            background: #d4edda;
            color: #155724;
        }
        .status.error {
            display: block;
            background: #f8d7da;
            color: #721c24;
        }
        .status.loading {
            display: block;
            background: #cce5ff;
            color: #004085;
        }
        .file-info {
            margin-top: 10px;
            padding: 8px;
            background: #f0f0f0;
            border-radius: 5px;
            font-size: 11px;
            display: none;
        }
        .file-info.show { display: block; }
        footer {
            text-align: center;
            padding: 10px;
            background: #f8f9fa;
            color: #666;
            font-size: 9px;
        }
        .warning-box {
            background: #fff3cd;
            border: 1px solid #ffc107;
            padding: 8px;
            margin-top: 15px;
            border-radius: 5px;
            font-size: 11px;
            text-align: center;
            color: #856404;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🏷️ DANFE Simplificada</h1>
            <p>Formato 10x15cm para etiquetas</p>
        </div>
        
        <div class="content">
            <div class="upload-area" id="uploadArea">
                <div style="font-size: 30px;">📁</div>
                <p>Clique ou arraste o XML da NF-e</p>
                <input type="file" id="fileInput" class="file-input" accept=".xml">
                <label for="fileInput" class="file-label">Selecionar XML</label>
            </div>
            
            <div id="fileInfo" class="file-info">
                <strong>📄 Arquivo:</strong> <span id="fileName"></span>
                <button onclick="removeFile()" style="float:right; background:#e74c3c; color:white; border:none; border-radius:10px; padding:2px 8px; cursor:pointer;">✖</button>
            </div>
            
            <button id="generateBtn" class="btn-generate" disabled>🏷️ Gerar DANFE</button>
            
            <div id="preview" class="preview">
                <h4 style="font-size:12px; margin-bottom:5px;">Pré-visualização</h4>
                <iframe id="pdfPreview" class="preview-iframe"></iframe>
                <button id="printBtn" class="btn-imprimir">🖨️ Imprimir Etiqueta</button>
            </div>
            
           
            
            <div id="status" class="status"></div>
        </div>
        
        <footer>
            <p>DANFE Simplificada - 100mm x 150mm | Sistema Licenciado</p>
        </footer>
    </div>
    
    <script>
        let selectedFile = null;
        
        const uploadArea = document.getElementById('uploadArea');
        const fileInput = document.getElementById('fileInput');
        const fileInfo = document.getElementById('fileInfo');
        const fileNameSpan = document.getElementById('fileName');
        const generateBtn = document.getElementById('generateBtn');
        const previewDiv = document.getElementById('preview');
        const pdfPreview = document.getElementById('pdfPreview');
        const printBtn = document.getElementById('printBtn');
        const statusDiv = document.getElementById('status');
        
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.style.background = '#e8f0fe';
        });
        
        uploadArea.addEventListener('dragleave', () => {
            uploadArea.style.background = '#f8f9fa';
        });
        
        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.style.background = '#f8f9fa';
            const files = Array.from(e.dataTransfer.files).filter(f => f.name.endsWith('.xml'));
            if (files.length > 0) selectFile(files[0]);
        });
        
        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length > 0) selectFile(e.target.files[0]);
        });
        
        function selectFile(file) {
            selectedFile = file;
            fileNameSpan.textContent = file.name;
            fileInfo.classList.add('show');
            generateBtn.disabled = false;
            previewDiv.classList.remove('show');
        }
        
        function removeFile() {
            selectedFile = null;
            fileInfo.classList.remove('show');
            generateBtn.disabled = true;
            previewDiv.classList.remove('show');
            fileInput.value = '';
        }
        
        generateBtn.addEventListener('click', async () => {
            if (!selectedFile) return;
            
            const formData = new FormData();
            formData.append('xml_file', selectedFile);
            
            generateBtn.disabled = true;
            statusDiv.className = 'status loading';
            statusDiv.innerHTML = '<p>⏳ Gerando DANFE...</p>';
            
            try {
                const response = await fetch('/gerar_danfe', {
                    method: 'POST',
                    body: formData
                });
                
                if (response.ok) {
                    const blob = await response.blob();
                    const url = URL.createObjectURL(blob);
                    pdfPreview.src = url;
                    previewDiv.classList.add('show');
                    
                    printBtn.onclick = () => {
                        const iframe = pdfPreview.contentWindow;
                        iframe.print();
                    };
                    
                    statusDiv.className = 'status success';
                    statusDiv.innerHTML = '<p>✅ DANFE gerada! Clique em imprimir.</p>';
                } else {
                    const error = await response.json();
                    throw new Error(error.error || 'Erro ao processar');
                }
            } catch (error) {
                statusDiv.className = 'status error';
                statusDiv.innerHTML = `<p>❌ Erro: ${error.message}</p>`;
            } finally {
                generateBtn.disabled = false;
            }
        });
    </script>
</body>
</html>
'''

def formatar_chave_espacada(chave):
    """Formata a chave em grupos de 4 dígitos"""
    if len(chave) == 44:
        return ' '.join([chave[i:i+4] for i in range(0, 44, 4)])
    return chave

def desenhar_linha_divisoria(c, x1, y, x2):
    """Desenha uma linha divisória pontilhada"""
    c.setStrokeColor(colors.HexColor('#999999'))
    c.setLineWidth(0.5)
    c.setDash(2, 2)
    c.line(x1, y, x2, y)
    c.setDash(1, 0)

def desenhar_texto_centralizado(c, texto, y, fonte, tamanho):
    """Desenha texto centralizado na página"""
    c.setFont(fonte, tamanho)
    largura_texto = c.stringWidth(texto, fonte, tamanho)
    largura_pagina = ETIQUETA_WIDTH
    x_central = (largura_pagina - largura_texto) / 2
    c.drawString(x_central, y, texto)

def desenhar_texto_esquerda(c, texto, y, fonte, tamanho, indent=0):
    """Desenha texto alinhado à esquerda"""
    c.setFont(fonte, tamanho)
    margem_esq = 3 * mm
    x = margem_esq + indent
    c.drawString(x, y, texto)

def extrair_dados_completos(xml_content):
    """Extrai dados completos da NF-e"""
    try:
        namespaces = {'nfe': 'http://www.portalfiscal.inf.br/nfe'}
        root = ET.fromstring(xml_content)
        
        def get_text(elemento, caminho):
            if elemento is None:
                return 'N/A'
            elem = elemento.find(caminho, namespaces)
            if elem is None:
                for e in elemento.iter():
                    tag = caminho.split('}')[-1] if '}' in caminho else caminho
                    if tag in e.tag:
                        return e.text if e.text else 'N/A'
                return 'N/A'
            return elem.text if elem.text else 'N/A'
        
        # Encontrar infNFe
        infNFe = None
        for elem in root.iter():
            if 'infNFe' in elem.tag:
                infNFe = elem
                break
        
        dados = {
            'chave_acesso': 'N/A',
            'emitente_nome': 'N/A',
            'emitente_cnpj': 'N/A',
            'emitente_ie': 'N/A',
            'emitente_uf': 'N/A',
            'destinatario_nome': 'N/A',
            'destinatario_cnpj': 'N/A',
            'destinatario_ie': 'N/A',
            'serie': 'N/A',
            'nNF': 'N/A',
            'dhEmi': 'N/A',
            'dhAutorizacao': 'N/A',
            'tpOper': '1',
            'vNF': '0.00',
            'vICMS': '0.00',
            'vIPI': '0.00',
            'vFrete': '0.00',
            'vDesc': '0.00',
            'protocolo': 'N/A',
            'natureza': 'N/A',
            'finalidade': 'N/A',
            'consumidor_final': 'N/A',
            'produtos': []
        }
        
        if infNFe is None:
            return dados
        
        # Chave
        chave = infNFe.get('Id', '')
        if chave and chave.startswith('NFe'):
            dados['chave_acesso'] = chave[3:]
        
        # Emitente
        emit = infNFe.find('.//nfe:emit', namespaces)
        if emit is not None:
            dados['emitente_nome'] = get_text(emit, './/nfe:xNome')
            dados['emitente_cnpj'] = get_text(emit, './/nfe:CNPJ')
            dados['emitente_ie'] = get_text(emit, './/nfe:IE')
            ender = emit.find('.//nfe:enderEmit', namespaces)
            if ender is not None:
                dados['emitente_uf'] = get_text(ender, './/nfe:UF')
        
        # Destinatário
        dest = infNFe.find('.//nfe:dest', namespaces)
        if dest is not None:
            dados['destinatario_nome'] = get_text(dest, './/nfe:xNome')
            dados['destinatario_cnpj'] = get_text(dest, './/nfe:CNPJ')
            if dados['destinatario_cnpj'] == 'N/A':
                dados['destinatario_cnpj'] = get_text(dest, './/nfe:CPF')
            dados['destinatario_ie'] = get_text(dest, './/nfe:IE')
            if dados['destinatario_ie'] == 'N/A':
                dados['destinatario_ie'] = 'Isento'
        
        # Dados NF
        ide = infNFe.find('.//nfe:ide', namespaces)
        if ide is not None:
            dados['serie'] = get_text(ide, './/nfe:serie')
            dados['nNF'] = get_text(ide, './/nfe:nNF')
            dh = get_text(ide, './/nfe:dhEmi')
            dados['dhEmi'] = dh[:10] if len(dh) > 10 else dh
            tp = get_text(ide, './/nfe:tpOper')
            dados['tpOper'] = 'SAÍDA' if tp == '1' else 'ENTRADA'
            dados['natureza'] = get_text(ide, './/nfe:natOp')
            dados['finalidade'] = get_text(ide, './/nfe:finNFe')
            fin_map = {'1': 'Normal', '2': 'Complementar', '3': 'Ajuste', '4': 'Devolução'}
            dados['finalidade'] = fin_map.get(dados['finalidade'], dados['finalidade'])
            dados['consumidor_final'] = get_text(ide, './/nfe:indFinal')
            consumidor_map = {'0': 'Não', '1': 'Sim'}
            dados['consumidor_final'] = consumidor_map.get(dados['consumidor_final'], dados['consumidor_final'])
        
        # Valores
        total = infNFe.find('.//nfe:total', namespaces)
        if total is not None:
            icms = total.find('.//nfe:ICMSTot', namespaces)
            if icms is not None:
                dados['vNF'] = get_text(icms, './/nfe:vNF')
                dados['vICMS'] = get_text(icms, './/nfe:vICMS')
                dados['vIPI'] = get_text(icms, './/nfe:vIPI')
                dados['vFrete'] = get_text(icms, './/nfe:vFrete')
                dados['vDesc'] = get_text(icms, './/nfe:vDesc')
        
        # Produtos
        produtos = []
        for det in infNFe.findall('.//nfe:det', namespaces)[:2]:
            prod = det.find('.//nfe:prod', namespaces)
            if prod is not None:
                try:
                    qtd = float(get_text(prod, './/nfe:qCom').replace(',', '.'))
                    vl_total = float(get_text(prod, './/nfe:vProd').replace(',', '.'))
                    vl_unit = vl_total / qtd if qtd > 0 else 0
                except:
                    qtd = 0
                    vl_total = 0
                    vl_unit = 0
                
                produto = {
                    'nome': get_text(prod, './/nfe:xProd')[:30],
                    'qtd': qtd,
                    'vl_unit': vl_unit,
                    'vl_total': vl_total
                }
                produtos.append(produto)
        dados['produtos'] = produtos
        
        # Protocolo e data de autorização
        prot = root.find('.//nfe:protNFe', namespaces)
        if prot is not None:
            inf = prot.find('.//nfe:infProt', namespaces)
            if inf is not None:
                dados['protocolo'] = get_text(inf, './/nfe:nProt')
                dh_auth = get_text(inf, './/nfe:dhRecibo')
                dados['dhAutorizacao'] = dh_auth[:10] if len(dh_auth) > 10 else dh_auth
        
        return dados
    except Exception as e:
        print(f"Erro: {e}")
        return {'chave_acesso': 'ERRO', 'emitente_nome': 'Erro', 'vNF': '0.00'}

def gerar_codigo_barras(c, chave, y, altura_mm=13):
    """
    Gera código de barras ocupando 90% da largura da página
    """
    # Largura disponível (90% da página)
    largura_disponivel = ETIQUETA_WIDTH * 0.9
    
    # Para Code 128, cada caractere tem 11 módulos
    # 44 caracteres = 484 módulos
    num_modulos = 44 * 11
    
    # Calcula largura do módulo
    module_width = largura_disponivel / num_modulos
    
    # Limita para valores legíveis (0.18 a 0.6 mm)
    if module_width < 0.18:
        module_width = 0.18
    elif module_width > 0.6:
        module_width = 0.6
    
    try:
        # Cria código de barras
        barcode = code128.Code128(
            chave,
            barHeight=altura_mm * mm,
            barWidth=module_width,
            quiet=False  # Remove margens laterais
        )
        
        # Centraliza horizontalmente
        largura_real = barcode.width
        x_central = (ETIQUETA_WIDTH - largura_real) / 2
        
        # Desenha
        barcode.drawOn(c, x_central, y - (altura_mm * mm) - 4)
        
        return altura_mm * mm + 8
        
    except Exception as e:
        print(f"Erro no código de barras: {e}")
        # Fallback
        barcode = code128.Code128(chave, barHeight=12*mm, barWidth=0.35)
        largura_real = barcode.width
        x_central = (ETIQUETA_WIDTH - largura_real) / 2
        barcode.drawOn(c, x_central, y - 12*mm - 4)
        return 12*mm + 8

def gerar_etiqueta_danfe(dados):
    """Gera PDF com código de barras otimizado"""
    buffer = io.BytesIO()
    c = canvas.Canvas(buffer, pagesize=(ETIQUETA_WIDTH, ETIQUETA_HEIGHT))
    
    width = ETIQUETA_WIDTH
    height = ETIQUETA_HEIGHT
    
    # Margens
    margem_esq = 3 * mm
    margem_dir = width - 3 * mm
    margem_bottom = 3 * mm
    
    # Posição Y inicial
    y = height - 3 * mm
    
    # Borda externa
    c.setStrokeColor(colors.black)
    c.setLineWidth(0.5)
    c.rect(margem_esq - 1, margem_bottom - 1, width - 4, height - 4, fill=0)
    
    # ===== TÍTULO =====
    desenhar_texto_centralizado(c, "DANFE SIMPLIFICADA - ETIQUETA", y - 5, "Helvetica-Bold", 10)
    y -= 12
    
    desenhar_linha_divisoria(c, margem_esq, y, margem_dir)
    y -= 8
    
    # ===== CHAVE DE ACESSO =====
    chave = dados.get('chave_acesso', '')
    if chave == 'N/A' or chave == 'ERRO':
        chave = '00000000000000000000000000000000000000000000'
    
    chave_espacada = formatar_chave_espacada(chave)
    
    desenhar_texto_centralizado(c, "CHAVE DE ACESSO", y - 4, "Helvetica", 6)
    y -= 8
    
    desenhar_texto_centralizado(c, chave_espacada, y - 2, "Helvetica-Bold", 6.5)
    y -= 12
    
    # ===== CÓDIGO DE BARRAS =====
    # Gera código ocupando 90% da largura (bem legível)
    y -= gerar_codigo_barras(c, chave, y, 13)
    
    # ===== CHAVE NUMÉRICA ABAIXO =====
    desenhar_texto_centralizado(c, chave_espacada, y - 2, "Helvetica", 5.5)
    y -= 10
    
    desenhar_linha_divisoria(c, margem_esq, y, margem_dir)
    y -= 8
    
    # ===== EMITENTE =====
    c.setFont("Helvetica-Bold", 8)
    desenhar_texto_centralizado(c, "EMITENTE", y - 4, "Helvetica-Bold", 8)
    y -= 9
    
    c.setFont("Helvetica", 6.5)
    nome_emit = dados.get('emitente_nome', 'N/A')
    if len(nome_emit) > 45:
        nome_emit = nome_emit[:42] + "..."
    desenhar_texto_esquerda(c, f"NOME: {nome_emit}", y - 3, "Helvetica", 6.5, 2)
    y -= 6
    
    desenhar_texto_esquerda(c, f"CNPJ: {dados.get('emitente_cnpj', 'N/A')}", y - 3, "Helvetica", 6.5, 2)
    y -= 6
    
    desenhar_texto_esquerda(c, f"IE: {dados.get('emitente_ie', 'N/A')}  UF: {dados.get('emitente_uf', 'N/A')}", y - 3, "Helvetica", 6.5, 2)
    y -= 8
    
    desenhar_texto_esquerda(c, f"SÉRIE: {dados.get('serie', 'N/A')}  N°: {dados.get('nNF', 'N/A')}  DATA: {dados.get('dhEmi', 'N/A')}", y - 3, "Helvetica", 6, 2)
    y -= 6
    
    desenhar_texto_esquerda(c, f"TIPO: {dados.get('tpOper', 'N/A')}", y - 3, "Helvetica", 6, 2)
    y -= 6
    
    natureza = dados.get('natureza', 'N/A')
    if len(natureza) > 48:
        natureza = natureza[:45] + "..."
    desenhar_texto_esquerda(c, f"NATUREZA: {natureza}", y - 3, "Helvetica", 5.5, 2)
    y -= 6
    
    desenhar_texto_esquerda(c, f"FINALIDADE: {dados.get('finalidade', 'N/A')}  |  CONSUMIDOR FINAL: {dados.get('consumidor_final', 'N/A')}", y - 3, "Helvetica", 5.5, 2)
    y -= 8
    
    desenhar_texto_esquerda(c, "PROTOCOLO DE AUTORIZAÇÃO", y - 3, "Helvetica-Bold", 5.5, 2)
    y -= 5
    desenhar_texto_esquerda(c, dados.get('protocolo', 'N/A'), y - 3, "Helvetica", 6, 4)
    y -= 5
    if dados.get('dhAutorizacao') != 'N/A':
        desenhar_texto_esquerda(c, f"AUTORIZADO EM: {dados.get('dhAutorizacao', 'N/A')}", y - 3, "Helvetica", 5, 4)
        y -= 5
    
    y -= 3
    desenhar_linha_divisoria(c, margem_esq, y, margem_dir)
    y -= 8
    
    # ===== DESTINATÁRIO =====
    c.setFont("Helvetica-Bold", 8)
    desenhar_texto_centralizado(c, "DESTINATÁRIO", y - 4, "Helvetica-Bold", 8)
    y -= 9
    
    c.setFont("Helvetica", 6.5)
    nome_dest = dados.get('destinatario_nome', 'N/A')
    if len(nome_dest) > 45:
        nome_dest = nome_dest[:42] + "..."
    desenhar_texto_esquerda(c, f"NOME: {nome_dest}", y - 3, "Helvetica", 6.5, 2)
    y -= 6
    
    cpf = dados.get('destinatario_cnpj', 'N/A')
    if len(cpf) == 11:
        cpf = f"{cpf[:3]}.{cpf[3:6]}.{cpf[6:9]}-{cpf[9:11]}"
    elif len(cpf) == 14:
        cpf = f"{cpf[:2]}.{cpf[2:5]}.{cpf[5:8]}/{cpf[8:12]}-{cpf[12:14]}"
    desenhar_texto_esquerda(c, f"CPF/CNPJ: {cpf}", y - 3, "Helvetica", 6.5, 2)
    y -= 6
    
    desenhar_texto_esquerda(c, f"IE: {dados.get('destinatario_ie', 'N/A')}", y - 3, "Helvetica", 6.5, 2)
    y -= 8
    
    # ===== PRODUTOS =====
    if dados.get('produtos') and len(dados['produtos']) > 0:
        c.setFont("Helvetica-Bold", 6.5)
        desenhar_texto_esquerda(c, "PRODUTOS", y - 3, "Helvetica-Bold", 6.5, 2)
        y -= 7
        
        for i, prod in enumerate(dados['produtos']):
            c.setFont("Helvetica-Bold", 6)
            nome_prod = prod['nome']
            if len(nome_prod) > 40:
                nome_prod = nome_prod[:37] + "..."
            desenhar_texto_esquerda(c, nome_prod, y - 3, "Helvetica-Bold", 6, 4)
            y -= 5
            
            c.setFont("Helvetica", 5.5)
            texto_info = f"   Qtd: {prod['qtd']:.2f}  x  R$ {prod['vl_unit']:.2f}  =  R$ {prod['vl_total']:.2f}".replace('.', ',')
            desenhar_texto_esquerda(c, texto_info, y - 3, "Helvetica", 5.5, 4)
            y -= 7
            
            if i < len(dados['produtos']) - 1:
                c.setStrokeColor(colors.HexColor('#cccccc'))
                c.setLineWidth(0.3)
                c.line(margem_esq + 2, y - 2, margem_dir - 2, y - 2)
                y -= 4
    
    desenhar_linha_divisoria(c, margem_esq, y, margem_dir)
    y -= 8
    
    # ===== RESUMO DOS VALORES =====
    try:
        valor = float(dados.get('vNF', '0').replace(',', '.'))
        vICMS = float(dados.get('vICMS', '0').replace(',', '.'))
        vFrete = float(dados.get('vFrete', '0').replace(',', '.'))
        vDesc = float(dados.get('vDesc', '0').replace(',', '.'))
    except:
        valor = vICMS = vFrete = vDesc = 0
    
    c.setFont("Helvetica-Bold", 8)
    desenhar_texto_centralizado(c, "RESUMO DOS VALORES", y - 4, "Helvetica-Bold", 8)
    y -= 9
    
    c.setFont("Helvetica", 7)
    desenhar_texto_esquerda(c, f"PRODUTOS: R$ {valor:.2f}".replace('.', ','), y - 3, "Helvetica", 7, 2)
    y -= 7
    
    if vFrete > 0:
        desenhar_texto_esquerda(c, f"FRETE: R$ {vFrete:.2f}".replace('.', ','), y - 3, "Helvetica", 7, 2)
        y -= 7
    
    if vDesc > 0:
        desenhar_texto_esquerda(c, f"DESCONTO: R$ {vDesc:.2f}".replace('.', ','), y - 3, "Helvetica", 7, 2)
        y -= 7
    
    desenhar_texto_esquerda(c, f"ICMS: R$ {vICMS:.2f}".replace('.', ','), y - 3, "Helvetica", 7, 2)
    y -= 10
    
    # ===== VALOR TOTAL =====
    c.setStrokeColor(colors.HexColor('#cccccc'))
    c.setLineWidth(0.5)
    c.line(margem_esq, y - 2, margem_dir, y - 2)
    y -= 6
    
    c.setFont("Helvetica-Bold", 9)
    c.setFillColor(colors.HexColor('#2c3e50'))
    texto_valor_label = "VALOR TOTAL:"
    largura_label = c.stringWidth(texto_valor_label, "Helvetica-Bold", 9)
    
    c.setFont("Helvetica-Bold", 11)
    c.setFillColor(colors.HexColor('#1a1a1a'))
    valor_texto = f"R$ {valor:.2f}".replace('.', ',')
    largura_valor = c.stringWidth(valor_texto, "Helvetica-Bold", 11)
    
    largura_total = largura_label + largura_valor + 5
    x_inicio = (width - largura_total) / 2
    
    c.setFont("Helvetica-Bold", 9)
    c.setFillColor(colors.HexColor('#2c3e50'))
    c.drawString(x_inicio, y - 4, texto_valor_label)
    
    c.setFont("Helvetica-Bold", 11)
    c.setFillColor(colors.HexColor('#1a1a1a'))
    c.drawString(x_inicio + largura_label + 5, y - 4, valor_texto)
    
    # ===== RODAPÉ =====
    c.setFillColor(colors.black)
    c.setFont("Helvetica", 4.5)
    rodape1 = "Consulta: https://www.nfe.fazenda.gov.br"
    largura_rodape1 = c.stringWidth(rodape1, "Helvetica", 4.5)
    x_rodape1 = (width - largura_rodape1) / 2
    c.drawString(x_rodape1, margem_bottom + 5, rodape1)
    
    rodape2 = f"Documento emitido por sistema autorizado - {datetime.now().strftime('%d/%m/%Y %H:%M')}"
    largura_rodape2 = c.stringWidth(rodape2, "Helvetica", 4)
    x_rodape2 = (width - largura_rodape2) / 2
    c.drawString(x_rodape2, margem_bottom + 2, rodape2)
    
    c.save()
    buffer.seek(0)
    return buffer

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/gerar_danfe', methods=['POST'])
def gerar_danfe():
    try:
        file = request.files.get('xml_file')
        
        if not file or file.filename == '':
            return jsonify({'error': 'Nenhum arquivo selecionado'}), 400
        
        xml_content = file.read()
        dados = extrair_dados_completos(xml_content)
        pdf_buffer = gerar_etiqueta_danfe(dados)
        
        return send_file(
            pdf_buffer,
            mimetype='application/pdf',
            as_attachment=False,
            download_name=f'DANFE_{dados.get("nNF", "000")}.pdf'
        )
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    print("=" * 70)
    print("🏷️ DANFE Simplificada - Etiqueta 10x15cm")
    print("=" * 70)
    print("✅ Servidor iniciado!")
    print("📱 Acesse: http://127.0.0.1:5000")
    print("📏 Tamanho: 100mm x 150mm (10x15cm)")
    print("=" * 70)
    print("🔧 CÓDIGO DE BARRAS CORRIGIDO (VERSÃO SIMPLIFICADA)")
    print("   ✅ Código ocupa 90% da largura da página")
    print("   ✅ Largura otimizada para leitura")
    print("   ✅ Centralizado automaticamente")
    print("   ✅ Chave numérica acima e abaixo do código")
    print("=" * 70)
    print("💰 VALOR TOTAL: Discreto e profissional")
    print("📦 PRODUTOS: Formatados em 2 linhas")
    print("🖨️ Impressão: Use o botão 'Imprimir Etiqueta'")
    print("=" * 70)
    app.run(debug=True, host='127.0.0.1', port=5000)
