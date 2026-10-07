# Tests — Zona Azul Digital

## 1. Parâmetros

Utilizar a variante definida em `constitution.md`:

- Tarifa: 600 centavos/hora.
- Fração: 30 minutos (300 centavos).
- Teto: 5000 centavos.
- Tolerância: 0 minuto.
- Porta: 8005.

## 2. Abertura de bilhete — UC1

`POST /bilhetes`

| Cenário | Resultado esperado |
|---|---|
| Placa `ABC1D23` | 201; `id`, `placa`, `entrada`, `status: "aberto"` |
| Placa ausente | 422; `placa_invalida` |
| Placa com 6 caracteres | 422; `placa_invalida` |
| Placa com 8 caracteres | 422; `placa_invalida` |
| Placa com letras minúsculas | 422; `placa_invalida` |
| `entrada` válida com fuso | 201; utilizar instante informado |
| `entrada` ausente | 201; utilizar instante atual |
| `entrada` inválida | 422; `entrada_invalida` |
| Placa com bilhete aberto | 409; `bilhete_em_aberto` |

## 3. Cobrança e tolerância — UC2 e UC7

`POST /bilhetes/{id}/encerramento`

| Duração | Frações | Valor esperado |
|---:|---:|---:|
| 0 min | 0 | 0 |
| 1 min | 1 | 300 |
| 29 min | 1 | 300 |
| 30 min | 1 | 300 |
| 31 min | 2 | 600 |
| 59 min | 2 | 600 |
| 60 min | 2 | 600 |
| 61 min | 3 | 900 |
| 89 min | 3 | 900 |
| 90 min | 3 | 900 |
| 91 min | 4 | 1200 |

**Regra:** arredondar as frações para cima. Como a tolerância é zero, toda duração positiva gera cobrança integral.

## 4. Teto de cobrança — UC2

| Duração | Valor sem teto | Valor esperado |
|---:|---:|---:|
| 480 min | 4800 | 4800 |
| 481 min | 5100 | 5000 |
| 600 min | 6000 | 5000 |

> [!WARNING]
> Aplicar o teto de 5000 centavos após calcular as frações.

## 5. Encerramento — UC2

| Cenário | Resultado esperado |
|---|---|
| Bilhete aberto | 200; `id`, `placa`, `entrada`, `saida`, `minutos`, `valor_centavos` |
| Bilhete inexistente | 404; `bilhete_nao_encontrado` |
| Bilhete já encerrado | 409; `bilhete_ja_encerrado` |

O campo `valor_centavos` deve ser um inteiro.

## 6. Cancelamento — UC5

`POST /bilhetes/{id}/cancelamento`

| Cenário | Resultado esperado |
|---|---|
| Bilhete aberto | 200; `status: "cancelado"` |
| Bilhete inexistente | 404; `bilhete_nao_encontrado` |
| Bilhete encerrado | 409; `bilhete_nao_aberto` |
| Bilhete cancelado | 409; `bilhete_nao_aberto` |

O cancelamento não gera cobrança, `saida` ou `valor_centavos`.

## 7. Uma vaga por placa — UC8

| Cenário | Resultado esperado |
|---|---|
| Placa sem bilhete aberto | Abertura permitida (201) |
| Placa com bilhete aberto | 409; `bilhete_em_aberto` |
| Bilhete anterior encerrado | Nova abertura permitida (201) |
| Bilhete anterior cancelado | Nova abertura permitida (201) |

## 8. Bilhetes ativos — UC3

`GET /bilhetes/ativos`

| Cenário | Resultado esperado |
|---|---|
| Bilhetes abertos | 200; incluir somente abertos |
| Bilhetes encerrados ou cancelados | Não incluir |
| Nenhum bilhete aberto | Array vazio |
| Múltiplos bilhetes abertos | Mais recentes primeiro |

## 9. Histórico por placa — UC6

`GET /bilhetes?placa=ABC1D23`

| Cenário | Resultado esperado |
|---|---|
| Placa com bilhetes de diferentes status | 200; incluir todos |
| Placa sem histórico | 200; array vazio |
| Múltiplos registros | Mais recentes primeiro |

## 10. Relatório diário — UC4

`GET /relatorios/diario?data=AAAA-MM-DD`

| Cenário | Resultado esperado |
|---|---|
| Data válida | 200; campos obrigatórios do relatório |
| Data inválida | 422; `data_invalida` |
| Bilhete encerrado na data | Incluir |
| Bilhete encerrado em outra data | Excluir |
| Bilhete aberto ou cancelado | Não contabilizar |
| Múltiplos encerramentos | Somar faturamento e calcular média |

Campos obrigatórios: `data`, `total_bilhetes`, `faturamento_centavos`, `tempo_medio_minutos`.

## 11. Arredondamento da média — UC4

| Durações (min) | Média | Esperado |
|---|---:|---:|
| 10 e 11 | 10,5 | 11 |
| 20 e 21 | 20,5 | 21 |
| 47 e 48 | 47,5 | 48 |
| 30 e 40 | 35 | 35 |

Arredondar médias terminadas em 0,5 para cima.

## 12. Conformidade do contrato

- Utilizar `valor_centavos` como inteiro.
- Não substituir o campo por `valor` em ponto flutuante.
- Respeitar os campos e códigos HTTP definidos no contrato oficial.