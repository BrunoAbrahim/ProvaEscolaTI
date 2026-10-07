# Spec — Zona Azul Digital

## UC1 — Abrir bilhete

### Endpoint

`POST /bilhetes`

### Entrada obrigatória

```json
{
  "placa": "ABC1D23"
}
```

Também deve ser aceito o campo opcional `entrada`.

Quando `entrada` estiver presente:

- deve estar em ISO-8601 com fuso;
- o bilhete deve utilizar esse instante como entrada.

Quando `entrada` não estiver presente:

- deve ser utilizado o horário atual.

### Sucesso

**HTTP `201`**

A resposta deve conter:

- `id`;
- `placa`;
- `entrada`;
- `status: "aberto"`.

### Erros

#### Placa inválida

**HTTP `422`**

```json
{
  "erro": "placa_invalida"
}
```

#### Entrada inválida

**HTTP `422`**

```json
{
  "erro": "entrada_invalida"
}
```

#### Placa com bilhete já aberto

**HTTP `409`**

```json
{
  "erro": "bilhete_em_aberto"
}
```

### Critérios de aceite

- placa válida deve criar bilhete com HTTP `201`;
- bilhete criado deve possuir status `aberto`;
- entrada informada deve ser utilizada no bilhete;
- ausência de entrada deve utilizar o horário atual;
- placa inválida deve retornar HTTP `422`;
- entrada inválida deve retornar HTTP `422`;
- placa com bilhete aberto deve retornar HTTP `409`.

---

## UC2 — Encerrar bilhete

### Endpoint

`POST /bilhetes/{id}/encerramento`

### Processamento

O sistema deve:

1. localizar o bilhete;
2. verificar se está aberto;
3. determinar a saída;
4. calcular a duração em minutos;
5. aplicar a tolerância;
6. calcular o número de frações;
7. calcular o valor em centavos;
8. aplicar o teto diário;
9. alterar o estado do bilhete para `encerrado`.

### Cobrança

- cada fração possui `30 minutos`;
- cada fração custa `300 centavos`;
- o número de frações deve ser arredondado para cima;
- o valor máximo é de `5000 centavos`.

### Sucesso

**HTTP `200`**

A resposta deve conter:

- `id`;
- `placa`;
- `entrada`;
- `saida`;
- `minutos`;
- `valor_centavos`.

### Erros

#### Bilhete inexistente

**HTTP `404`**

```json
{
  "erro": "bilhete_nao_encontrado"
}
```

#### Bilhete já encerrado

**HTTP `409`**

```json
{
  "erro": "bilhete_ja_encerrado"
}
```

### Critérios de aceite

- bilhete aberto deve poder ser encerrado;
- `valor_centavos` deve ser inteiro;
- duração fracionada deve ser arredondada para cima;
- valor não pode ultrapassar `5000 centavos`;
- bilhete inexistente deve retornar HTTP `404`;
- bilhete já encerrado deve retornar HTTP `409`.

---

## UC3 — Listar bilhetes ativos

### Endpoint

`GET /bilhetes/ativos`

### Saída

**HTTP `200`**

Deve retornar um array contendo somente bilhetes com status `aberto`.

Os bilhetes mais recentes devem aparecer primeiro.

### Critérios de aceite

- somente bilhetes abertos devem aparecer;
- bilhetes encerrados não devem aparecer;
- bilhetes cancelados não devem aparecer;
- ausência de bilhetes abertos deve retornar array vazio;
- o resultado deve ser ordenado do mais recente para o mais antigo.

---

## UC4 — Relatório diário

### Endpoint

`GET /relatorios/diario?data=AAAA-MM-DD`

### Entrada

O parâmetro `data` deve utilizar exatamente o formato:

`AAAA-MM-DD`

### Saída

**HTTP `200`**

Formato esperado:

```json
{
  "data": "2026-10-05",
  "total_bilhetes": 12,
  "faturamento_centavos": 8400,
  "tempo_medio_minutos": 47
}
```

### Processamento

O relatório deve considerar apenas os bilhetes encerrados no dia consultado.

Deve calcular:

- quantidade de bilhetes encerrados;
- soma de `valor_centavos`;
- média dos minutos.

Quando a média resultar em `.5`, o valor deve ser arredondado para cima.

### Erro

#### Data inválida

**HTTP `422`**

```json
{
  "erro": "data_invalida"
}
```

### Critérios de aceite

- somente bilhetes encerrados no dia devem ser considerados;
- bilhetes cancelados não devem gerar faturamento;
- média terminada em `.5` deve ser arredondada para cima;
- data inválida deve retornar HTTP `422`.

---

## UC5 — Cancelar bilhete

### Endpoint

`POST /bilhetes/{id}/cancelamento`

### Processamento

Somente bilhetes com status `aberto` podem ser cancelados.

Um bilhete cancelado:

- recebe status `cancelado`;
- não possui cobrança;
- não gera `saida`;
- não gera `valor_centavos`.

### Sucesso

**HTTP `200`**

A resposta deve apresentar:

`status: "cancelado"`

### Erros

#### Bilhete inexistente

**HTTP `404`**

```json
{
  "erro": "bilhete_nao_encontrado"
}
```

#### Bilhete não aberto

**HTTP `409`**

```json
{
  "erro": "bilhete_nao_aberto"
}
```

### Critérios de aceite

- bilhete aberto pode ser cancelado;
- bilhete cancelado não gera cobrança;
- bilhete encerrado não pode ser cancelado;
- bilhete já cancelado não pode ser cancelado novamente;
- bilhete inexistente deve retornar HTTP `404`.

---

## UC6 — Histórico por placa

### Endpoint

`GET /bilhetes?placa=ABC1D23`

### Saída

**HTTP `200`**

Deve retornar todos os bilhetes da placa, independentemente do status.

Os registros mais recentes devem aparecer primeiro.

### Critérios de aceite

- bilhetes abertos devem aparecer;
- bilhetes encerrados devem aparecer;
- bilhetes cancelados devem aparecer;
- placa sem histórico deve retornar array vazio;
- registros devem ser ordenados do mais recente para o mais antigo.

---

## UC7 — Tolerância

A tolerância desta variante é de `0 minutos`.

Portanto:

- qualquer duração positiva gera cobrança;
- `1 minuto` já corresponde a uma fração de `30 minutos`;
- a cobrança começa desde o primeiro minuto.

### Critérios de aceite

| Duração | Valor esperado |
|---:|---:|
| 1 minuto | 300 centavos |
| 30 minutos | 300 centavos |
| 31 minutos | 600 centavos |

---

## UC8 — Uma vaga por placa

Uma placa não pode possuir dois bilhetes abertos simultaneamente.

### Critérios de aceite

- placa sem bilhete aberto pode criar um bilhete;
- placa com bilhete aberto deve receber HTTP `409`;
- após o encerramento do bilhete anterior, a placa pode abrir outro;
- após o cancelamento do bilhete anterior, a placa pode abrir outro.