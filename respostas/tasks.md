# Tasks — Zona Azul Digital

## Regra geral de execução

O desenvolvimento deve seguir TDD durante cada funcionalidade, e não somente após a implementação de todos os endpoints.

Para cada comportamento:

1. escrever ou preparar o teste correspondente;
2. executar o teste e confirmar a falha esperada;
3. implementar o comportamento mínimo necessário;
4. executar novamente os testes;
5. refatorar sem alterar o contrato;
6. seguir para o próximo comportamento.

---

## 1. Preparação do projeto

- Criar a aplicação utilizando Python e FastAPI.
- Configurar SQLite e pytest.
- Organizar as camadas `API → Service → Repository`.
- Configurar a aplicação para escutar na porta `8005`.
- Preparar o manifesto de dependências.
- Preparar a estrutura inicial da suíte de testes.

**Critério de conclusão:** estrutura preparada para implementar os endpoints e executar testes automatizados.

---

## 2. Configuração da variante

Configurar os parâmetros obrigatórios:

| Parâmetro | Valor |
|---|---:|
| `TARIFA_HORA_CENTAVOS` | 600 |
| `FRACAO_MINUTOS` | 30 |
| `TETO_DIARIO_CENTAVOS` | 5000 |
| `TOLERANCIA_MINUTOS` | 0 |
| `PORTA_SERVICO` | 8005 |

**Critério de conclusão:** todas as regras utilizam os valores da variante.

---

## 3. Modelo e persistência

Antes da implementação, preparar testes para armazenamento, recuperação e estados necessários aos casos de uso.

Implementar:

- dados dos bilhetes: `id`, `placa`, `entrada`, `saida`, `status`, `minutos` e `valor_centavos`;
- estados `aberto`, `encerrado` e `cancelado`;
- operações para salvar, localizar e atualizar bilhetes;
- consultas por ID, placa, status e data de encerramento;
- restrição de apenas um bilhete aberto por placa.

**Critério de conclusão:** testes da camada de persistência e das regras associadas passam.

---

## 4. Validações

Para cada validação, criar primeiro o teste correspondente.

Implementar:

- placa com exatamente 7 caracteres alfanuméricos maiúsculos;
- `entrada` opcional em ISO-8601 com fuso;
- `data` no formato `AAAA-MM-DD`;
- erros de validação previstos no contrato.

**Critério de conclusão:** entradas válidas e inválidas produzem exatamente os resultados previstos.

---

## 5. Abertura de bilhete — UC1 e UC8

**Endpoint:** `POST /bilhetes`

Aplicar TDD para:

- abertura com placa válida;
- entrada opcional válida;
- ausência de entrada;
- entrada inválida;
- placa inválida;
- conflito de placa com bilhete aberto.

Implementar:

- recebimento de `placa` e `entrada` opcional;
- utilização do instante informado ou, na ausência dele, do horário atual;
- bloqueio de dois bilhetes abertos para a mesma placa;
- criação com status `aberto`;
- HTTP `201` com `id`, `placa`, `entrada` e `status`;
- resposta de abertura com `entrada` em ISO-8601 com fuso `-03:00`.

**Critério de conclusão:** todos os testes de UC1 e UC8 previstos em `tests.md` passam.

---

## 6. Cálculo de cobrança — UC2 e UC7

Criar primeiro os testes para:

- 1 minuto;
- fração exata;
- minuto imediatamente adicional;
- tolerância zero;
- teto de cobrança.

Implementar:

- cálculo da duração em minutos;
- aplicação da tolerância;
- cálculo de frações de 30 minutos, arredondando para cima;
- cobrança de 300 centavos por fração na variante atual;
- teto de 5000 centavos por bilhete;
- valores monetários em centavos inteiros.

**Critério de conclusão:** os cenários numéricos definidos em `tests.md` passam.

---

## 7. Encerramento — UC2

**Endpoint:** `POST /bilhetes/{id}/encerramento`

Criar primeiro os testes para:

- encerramento válido;
- bilhete inexistente;
- bilhete já encerrado.

Implementar:

- localização do bilhete;
- verificação de estado;
- determinação da saída;
- cálculo da duração;
- cálculo da cobrança;
- alteração para status `encerrado`;
- resposta HTTP `200` com `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`;
- erros `404` e `409` previstos pelo contrato.

**Critério de conclusão:** encerramento válido e erros especificados passam na suíte.

---

## 8. Cancelamento — UC5

**Endpoint:** `POST /bilhetes/{id}/cancelamento`

Criar primeiro os testes para:

- cancelamento válido;
- bilhete inexistente;
- bilhete encerrado;
- bilhete já cancelado.

Implementar:

- cancelamento somente de bilhetes abertos;
- status `cancelado`;
- ausência de cobrança, `saida` e `valor_centavos`;
- HTTP `200` no sucesso;
- erros `404` e `409` previstos.

**Critério de conclusão:** todos os cenários de cancelamento de `tests.md` passam.

---

## 9. Consultas — UC3 e UC6

### Bilhetes ativos

**Endpoint:** `GET /bilhetes/ativos`

Criar testes para:

- somente abertos;
- exclusão de encerrados e cancelados;
- array vazio;
- ordenação.

Implementar a consulta correspondente.

### Histórico por placa

**Endpoint:** `GET /bilhetes?placa=ABC1D23`

Criar testes para:

- todos os status;
- placa sem histórico;
- ordenação.

Implementar a consulta correspondente.

**Critério de conclusão:** ambas as consultas retornam HTTP `200`, conteúdo e ordem previstos.

---

## 10. Relatório diário — UC4

**Endpoint:** `GET /relatorios/diario?data=AAAA-MM-DD`

Criar primeiro testes para:

- data válida;
- data inválida;
- filtro por data;
- exclusão de abertos e cancelados;
- total de bilhetes;
- faturamento;
- média;
- arredondamento de `0,5` para cima;
- cenário integrado definido em `tests.md`.

Implementar:

- validação de `data`;
- seleção apenas de bilhetes encerrados no dia;
- `total_bilhetes`;
- `faturamento_centavos`;
- `tempo_medio_minutos`;
- HTTP `200`;
- HTTP `422` com `data_invalida`.

**Critério de conclusão:** testes individuais e cenário integrado do relatório passam.

---

## 11. Tratamento de erros

Garantir exatamente:

| HTTP | Código |
|---|---|
| `422` | `placa_invalida` |
| `422` | `entrada_invalida` |
| `422` | `data_invalida` |
| `404` | `bilhete_nao_encontrado` |
| `409` | `bilhete_ja_encerrado` |
| `409` | `bilhete_nao_aberto` |
| `409` | `bilhete_em_aberto` |

Corpo:

```json
{"erro":"codigo"}
```

**Critério de conclusão:** todos os erros definidos retornam exatamente status e corpo previstos.

---

## 12. Execução e testes completos

Depois que os ciclos TDD individuais estiverem concluídos:

- executar toda a suíte pytest;
- verificar regressões;
- confirmar os casos de borda;
- corrigir falhas sem alterar o contrato.

**Critério de conclusão:** suíte completa passando.

---

## 13. Dockerfile e documentação

- Criar `Dockerfile`.
- Instalar as dependências necessárias.
- Utilizar `EXPOSE 8005`.
- Definir `CMD` para iniciar a aplicação.
- Garantir que o serviço escute na porta `8005`.
- Criar README com:
  - instalação;
  - execução local;
  - execução dos testes;
  - construção da imagem;
  - execução do container.

**Critério de conclusão:** aplicação pode ser iniciada e testada de maneira reproduzível seguindo apenas o README.

---

## 14. Segurança e higiene

- Não incluir senhas, tokens, chaves de API ou credenciais reais.
- Não adicionar segredos desnecessários.
- Evitar arquivos temporários e artefatos não necessários.

**Critério de conclusão:** repositório gerado não contém segredos nem arquivos indevidos.

---

## 15. Verificação final

- Conferir os oito casos de uso do `spec.md`.
- Validar os cenários de `tests.md`.
- Verificar os parâmetros da variante.
- Confirmar campos e códigos HTTP do contrato.
- Executar a suíte automatizada completa.
- Construir o Dockerfile.
- Confirmar `EXPOSE 8005`.
- Confirmar que `CMD` inicia a aplicação.
- Confirmar o serviço na porta `8005`.
- Conferir manifesto de dependências.
- Conferir README.
- Verificar ausência de segredos.

**Critério de conclusão:** implementação compatível com o contrato oficial e artefatos de SDLC exigidos, sem requisitos adicionais não autorizados.
