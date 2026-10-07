# Constitution — Zona Azul Digital

## 1. Objetivo

O sistema deve fornecer exclusivamente uma API REST para gerenciamento de bilhetes de estacionamento rotativo.

A API deve permitir:

- abrir bilhetes;
- encerrar bilhetes;
- cancelar bilhetes;
- listar bilhetes ativos;
- consultar histórico por placa;
- gerar relatório diário.

Não deve ser criado back-office ou interface gráfica.

---

## 2. Variante obrigatória

A implementação deve utilizar obrigatoriamente:

- `TARIFA_HORA_CENTAVOS = 600`
- `FRACAO_MINUTOS = 30`
- `TETO_DIARIO_CENTAVOS = 5000`
- `TOLERANCIA_MINUTOS = 0`
- `PORTA_SERVICO = 8005`

O serviço deve escutar na porta `8005`.

---

## 3. Convenções gerais

- A API deve utilizar JSON conforme o contrato REST.
- Valores monetários devem ser sempre representados como números inteiros em centavos.
- Valores monetários nunca devem ser retornados como ponto flutuante.
- Datas e horários devem utilizar ISO-8601 conforme o contrato.
- O campo opcional `entrada` recebido pela API deve utilizar ISO-8601 com fuso.
- A resposta de abertura deve representar o campo `entrada` com o deslocamento `-03:00`, conforme o contrato.
- IDs devem identificar unicamente cada bilhete.
- O contrato REST possui prioridade sobre exemplos ilustrativos.
- O exemplo antigo com campo `valor` em ponto flutuante deve ser ignorado.

---

## 4. Estados de um bilhete

Um bilhete pode possuir somente um dos seguintes status:

- `aberto`;
- `encerrado`;
- `cancelado`.

Somente bilhetes com status `aberto` podem ser encerrados ou cancelados.

---

## 5. Placa

A placa deve possuir:

- exatamente 7 caracteres;
- somente caracteres alfanuméricos;
- somente letras maiúsculas quando houver letras.

Placa ausente ou inválida deve retornar:

**HTTP `422`**

```json
{"erro":"placa_invalida"}
```

---

## 6. Regra de abertura

Uma placa pode possuir no máximo um bilhete aberto simultaneamente.

Caso já exista um bilhete aberto para a placa:

**HTTP `409`**

```json
{"erro":"bilhete_em_aberto"}
```

Depois que o bilhete for encerrado ou cancelado, a placa pode abrir um novo bilhete.

---

## 7. Regra de cobrança

Cada fração possui `30 minutos`.

Uma hora possui duas frações.

Como a tarifa por hora é de `600 centavos`, uma fração de 30 minutos custa `300 centavos`.

O número de frações deve sempre ser arredondado para cima.

Exemplos:

- 1 minuto → 1 fração;
- 30 minutos → 1 fração;
- 31 minutos → 2 frações;
- 60 minutos → 2 frações;
- 61 minutos → 3 frações.

Os exemplos acima servem apenas para demonstrar a regra de arredondamento e não substituem os parâmetros da variante.

---

## 8. Tolerância

A variante possui:

`TOLERANCIA_MINUTOS = 0`

Portanto, qualquer duração positiva deve gerar cobrança.

A lógica deve respeitar a regra geral:

- duração menor ou igual à tolerância → `valor_centavos = 0`;
- ultrapassou a tolerância → cobrar desde o primeiro minuto;
- a tolerância não deve ser descontada da duração cobrada.

---

## 9. Teto diário

O valor de um bilhete nunca pode ultrapassar:

`5000 centavos`

> [!WARNING]
> O teto deve ser aplicado após o cálculo normal da cobrança pelas frações.

---

## 10. Cancelamento

Um bilhete cancelado:

- recebe status `cancelado`;
- não gera cobrança;
- não gera `saida`;
- não gera `valor_centavos`.

---

## 11. Ordenação

As seguintes consultas devem apresentar os bilhetes mais recentes primeiro:

- `GET /bilhetes/ativos`;
- `GET /bilhetes?placa=ABC1D23`.

---

## 12. Relatório diário

O relatório diário deve:

- considerar os bilhetes encerrados na data informada;
- calcular `total_bilhetes`;
- calcular `faturamento_centavos`;
- calcular `tempo_medio_minutos`;
- considerar somente os bilhetes encerrados no dia para o cálculo do tempo médio;
- arredondar o tempo médio com a regra de `0,5` para cima;
- retornar exatamente os campos definidos pelo contrato REST.

---

## 13. Erros obrigatórios

| Situação | HTTP | Corpo JSON |
|---|---:|---|
| Placa ausente ou inválida | `422` | `{"erro":"placa_invalida"}` |
| Entrada inválida | `422` | `{"erro":"entrada_invalida"}` |
| Data inválida | `422` | `{"erro":"data_invalida"}` |
| Bilhete inexistente | `404` | `{"erro":"bilhete_nao_encontrado"}` |
| Encerrar bilhete já encerrado | `409` | `{"erro":"bilhete_ja_encerrado"}` |
| Cancelar bilhete não aberto | `409` | `{"erro":"bilhete_nao_aberto"}` |
| Abrir bilhete com placa já ocupada | `409` | `{"erro":"bilhete_em_aberto"}` |

---

## 14. Prioridade do contrato

Todas as operações devem seguir os endpoints, métodos HTTP, status, campos e comportamentos definidos no enunciado oficial.

Nenhum exemplo ilustrativo pode substituir o contrato obrigatório.

---

# Pontos a conferir no enunciado

Esta seção registra ambiguidades do enunciado e não cria novas regras operacionais.

## Encerramento de bilhete cancelado

O contrato determina que somente bilhetes abertos podem ser encerrados, mas não especifica explicitamente qual erro deve ser retornado ao tentar encerrar um bilhete cancelado.

Por esse motivo, nenhum código HTTP adicional foi definido para essa situação.

## Escopo do fuso horário

O contrato exige ISO-8601 com fuso para o campo opcional `entrada` e apresenta `-03:00` na resposta de abertura.

O enunciado não determina de forma igualmente explícita o deslocamento de todos os demais campos temporais.

## Relatório sem bilhetes

O contrato não especifica explicitamente o valor de `tempo_medio_minutos` quando não existir nenhum bilhete encerrado na data consultada.

Nenhum valor adicional foi definido automaticamente para esse cenário.

## Precisão da duração

O contrato exige o campo `minutos`, mas não especifica como segundos ou frações de minuto devem ser tratados.

Nenhuma regra adicional de arredondamento temporal foi criada além do arredondamento da cobrança por `FRACAO_MINUTOS`.
