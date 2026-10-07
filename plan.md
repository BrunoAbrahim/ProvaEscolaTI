# Plan — Zona Azul Digital

## 1. Stack tecnológica

| Componente | Tecnologia |
|---|---|
| Backend | Python + FastAPI |
| Persistência | SQLite |
| Testes automatizados | pytest |
| Porta do serviço | `8005` |

**Justificativas:**
- **FastAPI:** permite desenvolver endpoints REST, validar entradas e controlar respostas HTTP.
- **SQLite:** oferece persistência sem exigir um servidor de banco de dados separado.
- **pytest:** permite automatizar testes de regras de negócio, endpoints e casos de borda.

As escolhas técnicas não podem modificar o contrato REST.

## 2. Arquitetura

A aplicação deve seguir a separação:

`API / Rotas → Service → Repository`

| Camada | Responsabilidade |
|---|---|
| API / Rotas | Receber requisições, validar formatos, interpretar parâmetros e retornar respostas HTTP |
| Service | Aplicar regras de negócio, calcular cobranças, controlar estados e identificar conflitos |
| Repository | Persistir e consultar bilhetes, históricos e dados necessários aos relatórios |

**Justificativa:** separar transporte HTTP, regras de negócio e persistência facilita manutenção, testes e consistência da implementação.

## 3. Persistência e consistência

A camada Repository deve permitir:
- Salvar e recuperar bilhetes por ID.
- Consultar bilhetes por placa e status.
- Listar bilhetes abertos.
- Recuperar registros encerrados para relatórios.
- Consultar o histórico completo de uma placa.

A implementação deve preservar a integridade dos dados e impedir que uma mesma placa possua dois bilhetes abertos simultaneamente, inclusive diante de requisições concorrentes.

Após encerramento ou cancelamento, deve ser possível abrir outro bilhete para a placa.

**Justificativa:** garantir consistência entre a persistência e a regra de uma vaga por placa.

## 4. Valores monetários e cobrança

Armazenar, processar e retornar valores monetários em centavos inteiros, sem utilizar ponto flutuante.

Parâmetros obrigatórios:
- `TARIFA_HORA_CENTAVOS = 600`
- `FRACAO_MINUTOS = 30`
- `TETO_DIARIO_CENTAVOS = 5000`
- `TOLERANCIA_MINUTOS = 0`

A cobrança deve:
1. Determinar a duração do estacionamento.
2. Verificar a tolerância gratuita.
3. Calcular a quantidade de frações, arredondando para cima.
4. Calcular o valor em centavos.
5. Aplicar o teto máximo de 5000 centavos.

Cada fração de 30 minutos corresponde a 300 centavos.

Como a tolerância é zero, qualquer duração positiva deve ser cobrada integralmente.

**Justificativa:** preservar o cálculo exigido no contrato sem erros de precisão monetária.

## 5. Relógio e datas

Na abertura:
- Se `entrada` for informada, validar ISO-8601 com fuso e utilizar o instante fornecido.
- Caso contrário, utilizar o instante atual.

No encerramento:
- Determinar o instante de saída.
- Calcular a duração entre entrada e saída.
- Retornar os campos temporais conforme o contrato.

A resposta de abertura deve apresentar `entrada` no formato ISO-8601 com fuso `-03:00`.

**Justificativa:** permitir testes de duração, frações e teto sem depender de espera em tempo real.

## 6. Estados e transições

Estados previstos:
- `aberto`
- `encerrado`
- `cancelado`

Transições permitidas:
- `aberto → encerrado`
- `aberto → cancelado`

Somente bilhetes abertos podem ser encerrados ou cancelados.

O cancelamento não deve gerar cobrança, `saida` ou `valor_centavos`.

**Justificativa:** impedir operações incompatíveis com o estado atual do bilhete.

## 7. Consultas e relatório diário

A implementação deve:
- Listar somente bilhetes abertos.
- Retornar o histórico completo por placa, independentemente do status.
- Ordenar ambas as consultas com os registros mais recentes primeiro.
- Produzir relatórios considerando os bilhetes encerrados na data solicitada.

O relatório deve calcular:
- `total_bilhetes`
- `faturamento_centavos`
- `tempo_medio_minutos`

A média deve ser arredondada com a regra de 0,5 para cima.

## 8. Validações e erros

A camada API deve validar formatos de entrada e parâmetros HTTP.

A camada Service deve identificar violações das regras de negócio.

Os erros devem ser convertidos para os códigos HTTP e corpos JSON estabelecidos no contrato.

Não alterar ou inventar códigos de erro já definidos pelo enunciado.

## 9. Estratégia de testes

Utilizar pytest para verificar:
- Abertura, encerramento e cancelamento.
- Validação de placa e entrada.
- Frações exatas e minuto imediatamente adicional.
- Tolerância zero.
- Cobrança em centavos inteiros.
- Aplicação do teto.
- Conflitos de placa e transições de estado.
- Listagem de ativos e histórico por placa.
- Relatório diário e arredondamento da média.
- Status HTTP e erros do contrato.

Utilizar os cenários numéricos e casos de borda documentados em `tests.md`.

**Justificativa:** identificar violações do contrato e das regras de negócio antes da entrega.

## 10. Container e documentação

A implementação gerada deve incluir:
- `Containerfile`
- Manifesto de dependências
- README com instruções de execução
- Testes automatizados

### Containerfile

Deve permitir:
- Construir a imagem da aplicação.
- Executar o serviço na porta `8005`.
- Instalar as dependências necessárias.

### README

Deve documentar:
- Instalação das dependências.
- Inicialização da API.
- Execução dos testes.
- Execução em container.

## 11. Prioridade do contrato

Todas as decisões técnicas devem respeitar os endpoints, métodos HTTP, campos, formatos, status e regras de negócio estabelecidos no enunciado oficial.

Em caso de incompatibilidade, o contrato REST prevalece.