# Automação n8n – Processamento de Anexo CSV + Cotação do Dólar

**Autor:** Cayo Durães  

Este fluxo do n8n automatiza o processamento de e-mails recebidos com anexos CSV.  
Ao receber um e-mail, o fluxo:

- Salva o anexo CSV na pasta configurada (local ou Google Drive);
- Consulta a cotação do dólar (USD → BRL) em tempo real;
- Responde automaticamente ao remetente confirmando o processamento do anexo e informando a cotação do dia;
- Caso não haja CSV, responde informando a ausência do arquivo.

---

## Estrutura do Fluxo

O fluxo é composto por 4 nós principais:

1. **Gmail Trigger**  
   - Dispara quando um novo e-mail chega na conta configurada.
   - Deve estar configurado com as credenciais do Gmail.
   - Opção “Download Attachments” habilitada para capturar anexos.

2. **HTTP Request**  
   - Consulta a API pública `https://open.er-api.com/v6/latest/USD` para obter a cotação do dólar.
   - Não requer autenticação.
   - Retorna o valor atual do USD em BRL.

3. **Code (JavaScript)**  
   - Extrai remetente do e-mail.
   - Verifica se há anexo CSV.
   - Monta as variáveis que serão usadas na resposta (e-mail de destino, nome do arquivo, cotação do dólar).
   - Configura o binário do arquivo para ser salvo.
   - Torna flexível a pasta de destino.

4. **Send Email (Gmail)**  
   - Envia a resposta ao remetente com as informações montadas no Code node.
   - O corpo do e-mail cita:
     - Nome do anexo processado (ou “nenhum anexo” se não houver);
     - Cotação do dólar atual (USD → BRL).

---

## Configuração

### 1. Credenciais de E-mail
- **Entrada:** Configure a conta do Gmail no nó **Gmail Trigger**.
- **Saída:** Configure a conta do Gmail no nó **Send Email**.
- Ambas podem usar credenciais diferentes, se desejado.

### 2. Pasta de Destino (USEI LOCAL)
- **Local (default):** `/tmp/n8n-data/arquivo.csv` dentro do contêiner Docker.
- **Google Drive:** Substitua o nó de gravação de arquivo local por um nó **Google Drive**:
  - Configure suas credenciais do Google Drive.
  - Defina a pasta de destino.
  - Conecte o binário do CSV no nó Google Drive.

### 3. Remetente de Saída
- Configurável no nó **Send Email** nas credenciais do Gmail.
- Pode ser a mesma conta ou outra.

---

## Comportamento sem CSV
- Se o e-mail não contiver um anexo CSV:
  - Nenhum arquivo é salvo.
  - O remetente recebe um e-mail informando a ausência do arquivo, mas ainda vê a cotação do dólar.

---

## Variáveis Principais no Code Node

| Campo        | Descrição                                     |
|--------------|-----------------------------------------------|
| `to`         | Endereço de e-mail do remetente               |
| `csvFileName`| Nome do arquivo CSV (ou `nenhum-anexo`)        |
| `dolar`      | Cotação do dólar atual (USD → BRL)             |
| `hasCsv`     | Booleano indicando se existe CSV ou não        |

---

## Exportação do Workflow
- Exporte o fluxo no n8n em **Settings > Workflows > Export**.
- Salve como `workflow_cayo_duraes.json`.

---

## Entrega Final
Envie:
- O **workflow exportado** (`workflow_cayo_duraes.json`);
- Este **README.md** (documentação do fluxo).

---

