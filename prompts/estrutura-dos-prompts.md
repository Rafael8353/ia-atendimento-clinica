# Como os prompts dos agentes foram estruturados

O texto dos prompts não é publicado porque tem informações da clínica: preços, campanhas, convênios, horários e equipe. Este documento mostra **a estrutura** e as **técnicas** usadas.

## Agente 1: atendimento de novos contatos

Prompt com cerca de 40 mil caracteres, organizado em seções:

| Seção | Para que serve |
|---|---|
| Identidade e objetivo | Quem é o agente e qual o resultado esperado (levar o contato até o agendamento). |
| Regras de contexto e prioridade | O que vale mais quando duas instruções conflitam. |
| Abertura: responder ao interesse antes de pedir identificação | Evita que pedir o nome vire uma barreira na primeira mensagem. |
| Continuidade | Aproveitar o que a pessoa já disse, sem repetir perguntas. |
| Conversa com SPIN adaptado | Perguntas de situação, problema, implicação e necessidade, de forma leve. |
| Identificação e pacientes existentes | Diferenciar quem já é paciente de quem é novo. |
| Valor da consulta, campanhas, preços e pagamento | Como e quando falar de valores, sem presumir capacidade financeira. |
| Dúvidas e objeções | Respostas para as objeções mais comuns. |
| Confirmação de interesse e encaminhamento | Quando passar o contato para o agendamento. |
| Encaminhamento humano | Quando a IA deve parar e chamar um atendente. |
| Recusas, pausas e retomadas | Respeitar o "não" e retomar sem ser insistente. |
| Áudios, imagens e falhas de leitura | O que fazer quando a mensagem não pôde ser lida. |
| Limites clínicos e situações sensíveis | A IA não faz diagnóstico e encaminha casos de dor ou urgência. |
| Dados da clínica | Informações fixas (fonte única de verdade, para a IA não inventar). |
| Estilo | Tom de WhatsApp: mensagens curtas, humanas e sem menus. |
| Contrato de saída | Formato JSON obrigatório (abaixo). |
| Contexto desta conversa | Histórico da memória e dados do lead, injetados a cada mensagem. |

O prompt termina com uma **autoconferência**: antes de responder, o modelo verifica se respondeu a todas as dúvidas, se respeitou recusas e se não inventou condições. Essa conferência não aparece para o paciente.

**Saída estruturada (JSON Schema):**

```json
{
  "resposta": "mensagem enviada ao contato",
  "nome_cliente": "nome confirmado ou vazio",
  "ja_e_paciente": "sim | nao | nao_perguntado_ainda",
  "encaminhar_humano": false
}
```

Os campos alimentam o fluxo do Make. `encaminhar_humano` pausa a IA e avisa a equipe. `ja_e_paciente` muda a rota do lead no CRM. `nome_cliente` atualiza o cadastro.

## Agente 2: recepção de pacientes

Prompt de triagem que identifica o **motivo do contato** e direciona ao setor certo:

1. Dor ou desconforto após um procedimento
2. Ajuste ou problema em prótese
3. Financeiro
4. Agendamento de consulta
5. Outras dúvidas

O prompt também traz confirmação, remarcação e cancelamento, horários, convênios, um **glossário de termos que os pacientes usam** (para entender como as pessoas falam de verdade), situações sensíveis e restrições.

**Saída estruturada:**

```json
{
  "nome_cliente": "",
  "resposta": "mensagem ao paciente",
  "encaminhar_humano": true,
  "destino": "financeiro | agendamento | clinico_dor | clinico_protese | atendimento_geral | nenhum"
}
```

O campo `destino` aciona uma rota diferente no Make para cada setor.

## Boas práticas aplicadas

- **Dados fixos centralizados** numa seção própria, para o modelo não inventar preços nem horários.
- **Saída em JSON Schema estrito**, para o fluxo do Make nunca quebrar com respostas fora do formato.
- **Encaminhamento humano explícito**: a IA sabe quando não deve responder.
- **Versionamento**: cada mudança no prompt gera um backup do cenário anterior antes de ir para produção.
