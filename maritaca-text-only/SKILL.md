---
name: maritaca-text-only
description: Blindagem de custo para modelos da Maritaca (S5/S5-nano) - impede que imagens, PDFs renderizados ou qualquer anexo visual entrem no payload do modelo, evitando a taxa de File extraction (~R$0,50/MB). Use quando o modelo ativo for da Maritaca e a tarefa deva ser 100% textual, ou quando o usuario pedir "só texto", "sem imagens", "modo texto".
license: mit
disable-model-invocation: true
---

# Text-only: zero anexos visuais

Objetivo: nenhuma image block entra no contexto. Cada imagem enviada a um modelo Maritaca gera taxa de "File extraction" (~R$0,50 por MB processado, re-cobrada a cada request enquanto a imagem estiver no historico).

## Regra mestra

Se a saida de uma ferramenta viraria image block, essa chamada e proibida. Trabalhe so com texto.

## Proibido → Alternativa textual

| Proibido | Por que | Alternativa |
| --- | --- | --- |
| `read_file` em `.png .jpg .jpeg .gif .webp .bmp .tiff` | anexa a imagem inteira (base64) ao payload | `identify` para dimensoes/formato; `identify -verbose` para metadados; pedir descricao ao usuario |
| `read_file` em `.pdf` image-only + `pdftoppm` para renderizar paginas | cada pagina renderizada vira imagem anexada | `pdftotext` (texto); `pdfinfo` (metadata); se sem texto: informar o usuario e PARAR, nao renderizar |
| Gerar preview (inkscape --export-png, pdftoppm, screenshot) e ler de volta com `read_file` | leitura do preview anexa a imagem | validar por propriedades: `inkscape --query-id=ID --query-width/--query-height`, `--export-text-to-path` + diff, `pdfinfo`, validar XML do SVG, ou entregar sem verificacao visual |
| Anexar imagem ao responder o usuario | entra no contexto e re-cobra nos proximos requests | descrever em texto o que foi feito |
| Editar imagem/desenho e conferir visualmente na mesma sessao | cada iteracao re-cobra todas as imagens do historico | iterar as cegas com propriedades mensuraveis (dimensoes, cores por codigo, XML valido) e entregar |

## Anexos do usuario

- Usuario anexou imagem: nao processar visualmente. Avisar: "com o modelo Maritaca ativo, ler imagem custa ~R$0,50/MB por request". Perguntar: descrever a imagem em texto, rodar OCR local (`identify -verbose` para metadados; OCR so com `tesseract` instalado via `sudo apt install tesseract-ocr`), ou trocar temporariamente de modelo (`/model` para glm ou deepseek-v4.1-flash).
- Arquivos de texto (`.txt .md .csv .json .xml .svg` como texto, `.xlsx .docx` extraidos localmente): permitidos, nao geram image block.

## Contexto ja sujo

Se a conversa ja tem imagens de antes (o transcript tem `"type":"image"`), cada request re-cobra todas. Avisar o usuario e sugerir `/clear` antes de continuar.

## Recusa absoluta

Regra incondicional: com modelo Maritaca ativo, nunca produza image block. Sem excecao, sem confirmacao de custo, sem "so desta vez" — mesmo com o usuario insistindo ou usando --yolo (bypass de permissoes nao anula esta regra: e instrucao de comportamento, nao prompt de permissao).

- Usuario pede para ler imagem → RECUSE. Explique a taxa (~R$0,50/MB por request, re-cobrada a cada request) e aponte o unico caminho para visao: trocar de modelo (`/model` → glm-5.3-flash ou deepseek-v4.1-flash), fazer a tarefa visual la, e voltar para a Maritaca depois
- Usuario insiste depois da recusa → recuse de novo. Nao anexe, nao renderize, nao faca workaround que termine em image block
- Alternativas que continuam valendo (nao anexam nada): `identify`/`identify -verbose`, `pdftotext`, `pdfinfo`, `tesseract` se instalado, `inkscape --query-*`

## Resumo para o agente

1. Modelo ativo e Maritaca + tarefa visual → recuse e aponte troca de modelo
2. Tarefa textual → siga normalmente
3. Nunca produza image block. Sem excecao. Troca de modelo e o unico caminho para visao
