# Plan — Zona Azul Digital

## 1. Stack tecnológica

| Componente | Tecnologia |
|---|---|
| Backend | Python + FastAPI |
| Persistência | SQLite |
| Testes automatizados | pytest |
| Porta do serviço | `8005` |

### Justificativas

- **FastAPI:** permite desenvolver endpoints REST, validar entradas e controlar respostas HTTP.
- **SQLite:** oferece persistência sem exigir um servidor de banco de dados separado.
- **pytest:** permite automatizar testes de regras de negócio, endpoints e casos de borda.

As escolhas técnicas não podem modificar o contrato REST.

---

## 2. Arquitetura

A aplicação deve seguir a separação:

`API / Rotas → Service → Repository`

| Camada | Responsabilidade |
|---|---|
| API / Rotas | Receber requisições, validar formatos, interpretar parâmetros e retornar respostas HTTP |
| Service | Aplicar regras de negócio, calcular cobranças, controlar estados e identificar conflitos |
| Repository | Persistir e consultar bilhetes, históricos e dados necessários aos relatórios |

**Justificativa:** separar transporte HTTP, regras de negócio e persistência facilita manutenção, testes e consistência da implementação.

---

## 3. Persistência e consistência

A camada Repository deve permitir:

- salvar e recuperar bilhetes por ID;
- consultar bilhetes por placa e status;
- listar bilhetes abertos;
- recuperar registros encerrados para relatórios;
- consultar o histórico completo de uma placa.

A implementação deve preservar a regra de que uma mesma placa não pode possuir dois bilhetes abertos simultaneamente.

Após encerramento ou cancelamento, deve ser possível abrir outro bilhete para a placa.

**Justificativa:** garantir consistência entre a persistência e a regra de uma vaga por placa.

---

## 4. Valores monetários e cobrança

Armazenar, processar e retornar valores monetários em centavos inteiros, sem utilizar ponto flutuante.

Parâmetros obrigatórios:

- `TARIFA_HORA_CENTAVOS = 600`
- `FRACAO_MINUTOS = 30`
- `TETO_DIARIO_CENTAVOS = 5000`
- `TOLERANCIA_MINUTOS = 0`

A cobrança deve:

1. determinar a duração do estacionamento;
2. verificar a tolerância gratuita;
3. calcular a quantidade de frações, arredondando para cima;
4. calcular o valor em centavos;
5. aplicar o teto máximo de 5000 centavos.

Cada fração de 30 minutos corresponde a 300 centavos.

Como a tolerância é zero, qualquer duração positiva deve ser cobrada integralmente.

**Justificativa:** preservar o cálculo exigido no contrato sem erros de precisão monetária.

---

## 5. Relógio e datas

Na abertura:

- se `entrada` for informada, validar ISO-8601 com fuso e utilizar o instante fornecido;
- caso contrário, utilizar o instante atual;
- a resposta de abertura deve apresentar `entrada` em ISO-8601 com fuso `-03:00`.

No encerramento:

- determinar o instante de saída;
- calcular a duração entre entrada e saída;
- retornar os campos temporais conforme o contrato.

**Justificativa:** permitir testes de duração, frações e teto sem depender de espera em tempo real.

---

## 6. Estados e transições

Estados previstos:

- `aberto`;
- `encerrado`;
- `cancelado`.

Transições permitidas:

- `aberto → encerrado`;
- `aberto → cancelado`.

Somente bilhetes abertos podem ser encerrados ou cancelados.

O cancelamento não deve gerar cobrança, `saida` ou `valor_centavos`.

**Justificativa:** impedir operações incompatíveis com o estado atual do bilhete.

---

## 7. Consultas e relatório diário

A implementação deve:

- listar somente bilhetes abertos;
- retornar o histórico completo por placa, independentemente do status;
- ordenar ambas as consultas com os registros mais recentes primeiro;
- produzir relatórios considerando os bilhetes encerrados na data solicitada.

O relatório deve calcular:

- `total_bilhetes`;
- `faturamento_centavos`;
- `tempo_medio_minutos`.

A média deve ser arredondada com a regra de `0,5` para cima.

---

## 8. Validações e erros

A camada API deve validar formatos de entrada e parâmetros HTTP.

A camada Service deve identificar violações das regras de negócio.

Os erros devem ser convertidos para os códigos HTTP e corpos JSON estabelecidos no contrato.

Não alterar ou inventar códigos de erro já definidos pelo enunciado.

---

## 9. Estratégia de testes

Utilizar pytest para verificar:

- abertura, encerramento e cancelamento;
- validação de placa e entrada;
- frações exatas e minuto imediatamente adicional;
- tolerância zero;
- cobrança em centavos inteiros;
- aplicação do teto;
- conflitos de placa e transições de estado;
- listagem de ativos e histórico por placa;
- relatório diário e arredondamento da média;
- status HTTP e erros do contrato.

Utilizar os cenários numéricos e casos de borda documentados em `tests.md`.

### Estratégia TDD

O ciclo TDD deve ocorrer durante a implementação de cada funcionalidade:

1. escrever ou preparar o teste do comportamento;
2. executar o teste e confirmar a falha esperada;
3. implementar apenas o comportamento necessário;
4. executar novamente os testes;
5. refatorar sem alterar o contrato;
6. repetir o processo para o próximo comportamento.

**Justificativa:** identificar violações do contrato e das regras de negócio durante a implementação, e não apenas ao final.

---

## 10. Execução reproduzível

A implementação gerada deve possuir uma forma reproduzível de instalação, inicialização e execução dos testes.

### Execução local

O README deve documentar, de forma completa:

- instalação das dependências;
- inicialização da API;
- porta utilizada (`8005`);
- execução dos testes automatizados.

### Dockerfile

A implementação deve incluir `Dockerfile`.

O Dockerfile deve:

- instalar as dependências necessárias;
- utilizar `EXPOSE 8005`;
- possuir `CMD` para iniciar a aplicação;
- iniciar o serviço escutando na porta `8005`.

A aplicação containerizada não deve depender de ajustes manuais posteriores à construção da imagem.

### Testes

O README deve informar o comando necessário para executar a suíte automatizada.

A mesma suíte deve ser executável de maneira previsível no ambiente documentado.

---

## 11. Segurança e higiene do repositório

Não devem ser incluídos no repositório:

- senhas;
- tokens;
- chaves de API;
- credenciais reais;
- outros segredos.

Caso alguma configuração sensível fosse necessária, ela deveria ser fornecida externamente, e não versionada.

Para esta aplicação, não devem ser introduzidos segredos sem necessidade.

---

## 12. Verificações básicas de qualidade

Antes da entrega:

- executar os testes automatizados;
- confirmar que a aplicação inicia na porta `8005`;
- verificar se o Dockerfile constrói e inicia o serviço;
- conferir se os endpoints respeitam o contrato;
- conferir se não existem valores monetários em ponto flutuante;
- verificar se o repositório gerado contém os artefatos exigidos;
- evitar arquivos temporários, credenciais e conteúdo não necessário.

---

## 13. Artefatos da implementação gerada

A implementação deve incluir:

- `Dockerfile`;
- manifesto de dependências;
- README com instruções de execução;
- testes automatizados.

---

## 14. Prioridade do contrato

Todas as decisões técnicas devem respeitar os endpoints, métodos HTTP, campos, formatos, status e regras de negócio estabelecidos no enunciado oficial.

Em caso de incompatibilidade, o contrato REST prevalece.

---

# Pontos a conferir no enunciado

As ambiguidades registradas em `constitution.md` e `spec.md` não devem ser resolvidas por decisões técnicas inventadas neste plano.
