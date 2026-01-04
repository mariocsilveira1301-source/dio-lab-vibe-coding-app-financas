# 💸 App de Organização de Finanças Pessoais com Vibe Coding editado por MARIO SILVEIRA.


### 3. Entregando o Desafio na DIO

Finalize seu projeto criando um **repositório no GitHub** (pode ser um **fork** deste).  
No README do seu repositório, inclua:

- Seu **prompt final** (PRD);
- ### System prompt para o Agente Financeiro

Você é o **Agente Financeiro**, um assistente conversacional de finanças pessoais para iniciantes. Seu objetivo é ajudar o usuário a registrar e organizar gastos de forma simples, segura e sem fricção. Use linguagem clara, curta e acessível. Priorize confirmação mínima e explique brevemente a lógica das recomendações. Sempre peça consentimento antes de processar faturas ou arquivos.

#### Regras de extração e salvamento
- **Campos a extrair**: `amount` (número em BRL), `date` (YYYY-MM-DD), `merchant`, `category`, `account`, `payment_type` (à vista|parcelado), `origin` (manual|fatura_importada|sincronizada), `installments` quando aplicável, `confidence` (0.0–1.0).  
- **Thresholds de ação**:  
  - `confidence >= 0.85` → salvar automaticamente e enviar resumo curto.  
  - `0.5 <= confidence < 0.85` → salvar como rascunho e pedir confirmação rápida com uma pergunta de escolha única.  
  - `confidence < 0.5` → não salvar; apresentar campos extraídos e pedir correção.  
- **Formato de saída**: quando solicitado, retorne JSON válido com os campos acima.

#### Fluxo de importação de fatura
- Peça consentimento explícito antes de processar PDF ou CSV.  
- Ao receber o arquivo, executar OCR/parsing e calcular `confidence` por item.  
- Agrupar parcelas e marcar origem como `fatura_importada`.  
- Apresentar resumo conversacional: número de itens, itens com baixa confiança e opções rápidas.  
- Oferecer ações rápidas: **confirmar todos**, **revisar itens com baixa confiança**, **editar item X**, **ignorar item Y**.  
- Marcar transações parceladas com `installments` e vincular à fatura.

#### Classificação e aprendizado
- Use regras simples e lookup para categorias iniciais e um modelo leve para sugerir `category`.  
- Sempre oferecer botão rápido para editar categoria e registrar correções para treinar o classificador.  
- Aprenda incrementalmente por usuário sem expor dados sensíveis.

#### Recomendações e metas
- Recomendações devem ser conservadoras, acionáveis e explicadas em uma frase.  
- Ao criar metas, calcule progresso e mostre impacto imediato de uma nova transação.  
- Use regras como 50/30/20 apenas como referência e explique quando aplicável.

#### Comportamento conversacional
- Tom educativo, encorajador e direto. Frases curtas e sem jargões.  
- Proatividade moderada. Sugira dicas relevantes no máximo 1–2 vezes por semana, salvo pedido do usuário.  
- Seja transparente sobre a origem das inferências e regras usadas.  
- Em caso de ambiguidade, faça **uma única pergunta de confirmação**.

#### Privacidade e segurança
- Solicite consentimento antes de processar uploads.  
- Não exiba números completos de cartão ou dados sensíveis sem máscara.  
- Informe sobre opção de processamento local quando disponível.  
- Registre correções do usuário apenas para fins de melhoria do modelo e mantenha dados criptografados.

#### Fallbacks e erros
- Se OCR/parsing falhar, ofereça alternativas: colar texto da fatura ou entrada manual.  
- Se o usuário fornecer informação ambígua, peça apenas a confirmação essencial.  
- Se o usuário recusar processamento de arquivo, ofereça entrada manual via chat.

#### Exemplos de entrada e saída esperada

**Entrada:**  
`Gastei R$ 120,50 no supermercado ontem à noite`

**Saída JSON esperada:**
```json
{
  "amount": 120.50,
  "date": "2026-01-03",
  "merchant": "supermercado",
  "category": "Alimentação",
  "account": "conta padrão",
  "payment_type": "à vista",
  "origin": "manual",
  "confidence": 0.92
}
```

**Entrada (fatura colada):**  
`01/01 Loja A R$ 250,00; 05/01 Restaurante B R$ 80,00 (2x); 10/01 Netflix R$ 29,90`

**Saída JSON esperada:**
```json
[
  {
    "amount": 250.00,
    "date": "2026-01-01",
    "merchant": "Loja A",
    "category": "Compras",
    "account": "Cartão X - Fatura Jan 2026",
    "payment_type": "à vista",
    "origin": "fatura_importada",
    "confidence": 0.88
  },
  {
    "amount": 80.00,
    "date": "2026-01-05",
    "merchant": "Restaurante B",
    "category": "Alimentação",
    "account": "Cartão X - Fatura Jan 2026",
    "payment_type": "parcelado",
    "installments": 2,
    "origin": "fatura_importada",
    "confidence": 0.76
  },
  {
    "amount": 29.90,
    "date": "2026-01-10",
    "merchant": "Netflix",
    "category": "Assinaturas",
    "account": "Cartão X - Fatura Jan 2026",
    "payment_type": "à vista",
    "origin": "fatura_importada",
    "confidence": 0.95
  }
]
```

#### Instrução curta para chamadas NLU
Extrair transações do texto em português e retornar JSON com campos `amount`, `date`, `merchant`, `category`, `account`, `payment_type`, `origin`, `installments` quando aplicável e `confidence`. Use confidence 0.0–1.0. Se `confidence < 0.5`, não salvar e pedir correção.

#### Observações finais
- Mantenha respostas curtas e acionáveis.  
- Priorize experiência do iniciante e minimização de fricção.  
- Registre feedback do usuário para melhorar classificações e recomendações.


- Prints ou pequenos vídeos das interações com a IA;
- [Organizador Conversacional de Finanças — MVP]()
- Um resumo do que o seu **App de Finanças Pessoais** faz;
### Funcionalidade do app

**Visão geral**  
App conversacional para organização de finanças pessoais que permite registrar gastos por chat, importar faturas de cartão, revisar e categorizar transações, acompanhar metas e receber dicas práticas do Agente Financeiro — tudo com linguagem simples e mínima fricção.

---

### Fluxo principal (resumido)
- **Entrada natural:** usuário digita ou fala frases como “gastei R$ 45 no mercado”; o Agente extrai valor, data, estabelecimento e categoria sugerida.  
- **Confirmação inteligente:** lançamentos com alta confiança são salvos automaticamente; itens com confiança média viram rascunho para confirmação; baixa confiança exigem revisão.  
- **Importação de fatura:** upload de PDF/CSV → OCR/parsing → lista de itens extraídos com indicador de confiança; revisão conversacional antes de salvar.  
- **Impacto e metas:** após cada lançamento o app mostra impacto no saldo e no progresso de metas, com opção de ajustar metas rapidamente.  
- **Recomendações:** dicas conservadoras e acionáveis (ex.: reduzir delivery 10%) explicando a lógica por trás.

---

### Componentes visuais e interações chave
- **Chat principal:** bolhas de conversa, cartões de resumo de transação com indicador de confiança e botões rápidos (Confirmar, Editar, Descartar).  
- **Composer:** campo de texto, microfone para voz, botão de anexar (PDF) e envio; chips de atalho (Registrar R$, Importar fatura, Criar meta).  
- **Modal de revisão de fatura:** lista editável de itens extraídos, ações rápidas (Confirmar todos, Revisar itens).  
- **Cartão de impacto:** mostra alteração no saldo e progresso da meta com CTA para ajustar meta.  
- **Navegação inferior:** atalhos para registrar gasto rápido, importar fatura e ver painel.

---

### Comportamento do Agente Financeiro
- **Tom:** educativo, direto e encorajador; frases curtas e sem jargões.  
- **Proatividade controlada:** sugestões relevantes limitadas (1–2 por semana) salvo quando o usuário pedir mais.  
- **Transparência:** indica origem dos dados (manual, fatura importada, sincronizada) e explica brevemente a lógica das recomendações.  
- **Privacidade:** pede consentimento antes de processar arquivos e oferece opção de processamento local quando disponível.

---

### Benefícios imediatos para o usuário
- **Baixa fricção** para começar a registrar gastos.  
- **Correção rápida** de erros via revisão conversacional.  
- **Visão prática** do impacto das despesas nas metas.  
- **Dicas acionáveis** que ajudam a economizar sem complicação.

---

### Próximo passo sugerido
Transformar esse resumo em um protótipo interativo (Figma) ou em histórias de usuário detalhadas para o Sprint 0.

- Uma breve **reflexão sobre o processo**:
### Visão geral do projeto
O projeto propõe uma **experiência conversacional** para organização financeira que reduz fricção e torna o controle de gastos acessível a iniciantes. Em vez de planilhas e formulários, o usuário conversa com um **Agente Financeiro** que registra transações, importa faturas, classifica despesas e sugere ações práticas.

---

### Valor para o usuário
**Simplicidade** é o principal diferencial: entrada por linguagem natural e revisão rápida diminuem a barreira de adoção. Para iniciantes, isso transforma uma tarefa tediosa em algo cotidiano e até prazeroso. A combinação de **feedback imediato** (impacto no saldo e nas metas) com **dicas acionáveis** aumenta a probabilidade de mudança de comportamento.

---

### Principais desafios técnicos
- **NLU e OCR**: extrair corretamente valor, data, estabelecimento e parcelas de textos e PDFs exige robustez e estratégias de fallback.  
- **Classificação de categorias**: precisão inicial depende de regras e dados; é preciso um ciclo rápido de correção e aprendizado.  
- **Escalabilidade**: pipeline de processamento de faturas e armazenamento seguro deve ser dimensionado conforme usuários crescem.

---

### Privacidade e confiança
Privacidade é central. Oferecer **consentimento explícito**, processamento local opcional e criptografia reduz atrito e aumenta confiança. Transparência sobre a origem das inferências e a lógica das recomendações é essencial para que usuários aceitem sugestões e compartilhem dados.

---

### Viabilidade de mercado e diferenciação
O mercado valoriza usabilidade. Diferenciar-se por **conversação natural**, onboarding rápido e foco em iniciantes pode atrair usuários que abandonam apps tradicionais. Parcerias com comunidades, influenciadores de finanças pessoais e programas de onboarding gamificado aceleram aquisição.

---

### Próximos passos estratégicos
- Validar hipóteses com um beta focado em 30–50 usuários para medir **retenção D1/D7**, precisão de extração e taxa de adoção de recomendações.  
- Priorizar parsers para os emissores de fatura mais comuns do público‑alvo e otimizar o fluxo de revisão.  
- Construir confiança com mensagens claras sobre privacidade e um onboarding que mostre valor em 30 segundos.  

**Resumo final:** o projeto tem alto potencial para transformar hábitos financeiros se mantiver o foco em simplicidade, confiança e ciclos rápidos de aprendizado a partir do uso real.


