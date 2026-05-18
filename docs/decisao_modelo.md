# Decisão de Modelo — BluaDiagnostics
## Care Plus | Plataforma Blua
## Sprint 1 — PoC Acadêmica FIAP
## Versão: 1.0.0 | 2026-05-15

---

## 1. Contexto da Decisão

A escolha do modelo de linguagem é a decisão técnica mais
crítica do BluaDiagnostics. O agente lida com contexto clínico
cardiovascular — qualquer falha em seguir restrições pode
resultar em orientação inadequada ao beneficiário.

Os critérios avaliados foram definidos pelo challenge:
**latência, custo, privacidade/LGPD e qualidade clínica.**

Dois modelos foram avaliados em profundidade:
- **Qwen qwen-plus** (Alibaba — via DashScope International)
- **Llama 3.3 70B** (Meta — via Groq API ou on-prem)

---

## 2. Tabela Comparativa — Critérios Obrigatórios

| Critério | Qwen qwen-plus (escolhido) | Llama 3.3 70B |
|---|---|---|
| **Latência** | ~800ms via DashScope cloud | >60s em CPU — GPU T4 necessária no Colab |
| **Custo** | Gratuito — 1M tokens grátis por 90 dias no DashScope free trial | Gratuito — mas exige GPU paga no Colab ou Groq com risco LGPD |
| **Privacidade / LGPD** | Dado transita pelo DashScope Internacional. Mitigação: modo Ollama on-prem disponível | Via Groq: dado transita fora do Brasil. On-prem: resolve LGPD mas inviável no Colab gratuito sem GPU |
| **Qualidade clínica** | IFBench 76,5 — instruction following superior, PT-BR nativo, 201 idiomas | IFBench ~71 — bom desempenho geral, sem foco clínico em português |

---

## 3. Tabela Comparativa — Critérios Técnicos Adicionais

| Critério | Qwen qwen-plus | Llama 3.3 70B |
|---|---|---|
| Lançamento | 2025–2026 | Dezembro/2024 |
| Arquitetura | Dense + MoE 35B-A3B | Dense 70B |
| Function calling | Nativo OpenAI-compatible | Suportado, requer mais reescrita |
| Hybrid thinking mode | Sim — toggle por chamada sem trocar modelo | Não disponível |
| Contexto máximo | Até 1M tokens | 128K tokens |
| Licença | Apache 2.0 — sem restrições comerciais | Llama Community License — restrições de uso comercial |
| Disponibilidade Colab CPU | Sim — inferência em cloud via DashScope | Não — exige GPU robusta para rodar on-prem |
| Frameworks de agente | `qwen-agent` oficial + LangGraph | LangGraph |

---

## 4. Análise por Critério

### Latência

O Qwen via DashScope responde em aproximadamente 800ms em
chamadas simples — viável para uma experiência conversacional
fluida no Colab. O Llama 3.3 70B exige GPU T4 para rodar
em tempo aceitável no Colab — sem GPU, o tempo de resposta
ultrapassa 60 segundos por turno, inviabilizando a demo.

**Vencedor: Qwen**

### Custo

Ambos são gratuitos dentro das limitações de cada plataforma.
O DashScope oferece 1 milhão de tokens gratuitos por 90 dias
sem necessidade de cartão de crédito para ativação do free
trial. O Groq oferece plano gratuito para Llama, mas com
rate limits mais restritivos e sem garantia de disponibilidade
de GPU no Colab.

**Empate — leve vantagem para Qwen pela estabilidade do free
trial DashScope.**

### Privacidade / LGPD

Nenhum dos dois resolve LGPD completamente no ambiente Colab —
dados transitam por servidores fora do Brasil em ambos os casos.
A diferença está na mitigação disponível:

O Qwen oferece modo on-prem via Ollama (`qwen:14b`) para o
projeto final — dado não sai da máquina, LGPD resolvida.
O Llama 3.3 70B on-prem exige hardware robusto (GPU com 40GB+
de VRAM) inviável para o contexto acadêmico atual.

Para a Sprint 1, ambos os modelos operam com dados mockados
sem PII real — o risco LGPD é mitigado na origem.

**Vencedor: Qwen — pela viabilidade real da migração on-prem.**

### Qualidade Clínica

O IFBench (Instruction Following Benchmark) mede a capacidade
do modelo de seguir instruções complexas — critério diretamente
relacionado à eficácia dos guardrails clínicos. Qwen 76,5 vs
Llama ~71.

Em contexto clínico, instruction following é mais importante
que capacidade criativa — o agente precisa respeitar restrições
invioláveis (não diagnosticar, não prescrever, escalar red flags)
mesmo sob pressão de jailbreak. Modelos com IFBench mais alto
tendem a manter restrições com maior consistência.

O suporte nativo a português com 201 idiomas no treinamento do
Qwen reduz alucinação terminológica em bulas e protocolos da
SBC em PT-BR.

**Vencedor: Qwen**

---

## 5. Decisão Final

**Modelo escolhido: Qwen qwen-plus via DashScope International**

Cinco motivos consolidados:

1. **Instruction following superior (IFBench 76,5)** — garante
   que guardrails clínicos sejam respeitados de forma consistente,
   inclusive sob tentativas de jailbreak.

2. **PT-BR nativo de qualidade clínica** — reduz alucinação
   terminológica em protocolos cardiovasculares e bulas em
   português brasileiro.

3. **Hybrid thinking mode** — permite `thinking=ON` nos agentes
   de triagem e suporte clínico (raciocínio mais profundo) e
   `thinking=OFF` no roteador (latência mínima) sem trocar de
   modelo.

4. **Compatível com Colab CPU** — toda inferência roda em cloud
   via DashScope, sem necessidade de GPU paga no Colab gratuito.

5. **Caminho claro para on-prem** — migração para `qwen:14b`
   via Ollama no projeto final resolve LGPD sem reescrever
   arquitetura — apenas troca o parâmetro `backend`.

---

## 6. Risco Documentado — LGPD

**Risco:** dados de saúde transitam pelo DashScope International
(servidores fora do Brasil) durante a Sprint 1.

**Mitigação na Sprint 1:** todos os dados utilizados são mockados
— nenhum dado real de beneficiário é processado. O risco LGPD
é mitigado na origem.

**Mitigação no projeto final:** modo Ollama on-prem com
`qwen:14b` rodando em servidor local — dado não sai da máquina,
conformidade LGPD garantida.

---

## 7. Modelos Descartados e Motivos

| Modelo | Motivo do descarte |
|---|---|
| Qwen 3.5 / 3.6 | Bugs de function calling em XML ainda instáveis em Ollama e llama.cpp (abril/2026). Promissor para versões futuras. |
| Llama 3.1 8B | Qualidade clínica inferior ao 3.3 70B. Descartado como modelo de comparação por ser faixa de tamanho diferente. |
| GPT-4o | Pago — fora das restrições do projeto acadêmico. |
| Gemini Pro | Sem free tier estável para function calling no momento da avaliação. |

---

*Documento elaborado para a Sprint 1 do BluaDiagnostics.*
*Decisões revisáveis nas próximas sprints conforme evolução
dos modelos e requisitos do projeto.*