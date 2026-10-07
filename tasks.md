# Tasks — Zona Azul Digital

## 1. Preparação do projeto

- Criar a aplicação utilizando Python e FastAPI.
- Configurar SQLite e pytest.
- Organizar as camadas `API → Service → Repository`.
- Configurar a aplicação para escutar na porta `8005`.
- Preparar o manifesto de dependências.

**Critério de conclusão:** estrutura preparada para implementar os endpoints e executar testes automatizados.

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

## 3. Modelo e persistência

- Definir os dados dos bilhetes: `id`, `placa`, `entrada`, `saida`, `status`, `minutos` e `valor_centavos`.
- Representar os estados `aberto`, `encerrado` e `cancelado`.
- Implementar operações para salvar, localizar e atualizar bilhetes.
- Permitir consultas por ID, placa, status e data de encerramento.
- Preservar a restrição de apenas um bilhete aberto por placa.

**Critério de conclusão:** dados armazenados e recuperados corretamente para as operações previstas.

## 4. Validações

- Validar placas com exatamente 7 caracteres alfanuméricos maiúsculos.
- Validar `entrada` opcional em ISO-8601 com fuso.
- Validar `data` no formato `AAAA-MM-DD`.
- Retornar os erros de validação previstos no contrato.

**Critério de conclusão:** entradas inválidas geram os códigos HTTP e corpos JSON exigidos.

## 5. Abertura de bilhete — UC1 e UC8

**Endpoint:** `POST /bilhetes`

- Receber `placa` e `entrada` opcional.
- Utilizar o instante informado ou, na ausência dele, o horário atual.
- Impedir dois bilhetes abertos simultaneamente para a mesma placa.
- Criar o bilhete com status `aberto`.
- Retornar HTTP `201` com `id`, `placa`, `entrada` e `status`.
- Respeitar o fuso `-03:00` na resposta de abertura.

**Critério de conclusão:** abertura válida, entrada opcional e conflito de placa atendem ao contrato.

## 6. Cálculo de cobrança — UC2 e UC7

- Calcular a duração em minutos.
- Aplicar a tolerância de zero minuto.
- Calcular frações de 30 minutos, arredondando para cima.
- Cobrar 300 centavos por fração.
- Aplicar o teto de 5000 centavos por bilhete.
- Retornar valores monetários em centavos inteiros.

**Critério de conclusão:** os resultados numéricos correspondem aos cenários de `tests.md`.

## 7. Encerramento — UC2

**Endpoint:** `POST /bilhetes/{id}/encerramento`

- Localizar o bilhete e verificar se está aberto.
- Determinar o instante de saída.
- Calcular duração e cobrança.
- Alterar o status para `encerrado`.
- Retornar HTTP `200` com `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`.
- Tratar bilhete inexistente (`404`) e já encerrado (`409`) conforme o contrato.

**Critério de conclusão:** encerramento válido e erros conhecidos retornam as respostas exigidas.

## 8. Cancelamento — UC5

**Endpoint:** `POST /bilhetes/{id}/cancelamento`

- Permitir cancelamento somente de bilhetes abertos.
- Alterar o status para `cancelado`.
- Não gerar cobrança, `saida` ou `valor_centavos`.
- Retornar HTTP `200` com status `cancelado`.
- Tratar bilhete inexistente (`404`) e não aberto (`409`).

**Critério de conclusão:** cancelamento válido não gera cobrança e operações inválidas respeitam o contrato.

## 9. Consultas — UC3 e UC6

### Bilhetes ativos

**Endpoint:** `GET /bilhetes/ativos`

- Retornar apenas bilhetes abertos.
- Ordenar os mais recentes primeiro.
- Retornar array vazio quando não houver registros.

### Histórico por placa

**Endpoint:** `GET /bilhetes?placa=ABC1D23`

- Consultar todos os bilhetes da placa, independentemente do status.
- Ordenar os mais recentes primeiro.
- Retornar array vazio quando não houver histórico.

**Critério de conclusão:** ambas as consultas retornam HTTP `200` com os registros e a ordenação previstos.

## 10. Relatório diário — UC4

**Endpoint:** `GET /relatorios/diario?data=AAAA-MM-DD`

- Validar o parâmetro `data`.
- Considerar somente bilhetes encerrados no dia solicitado.
- Calcular `total_bilhetes`.
- Calcular `faturamento_centavos`.
- Calcular `tempo_medio_minutos`, arredondando 0,5 para cima.
- Retornar HTTP `200` com os quatro campos do contrato.
- Retornar `422` com `data_invalida` quando necessário.

**Critério de conclusão:** os valores do relatório são calculados conforme o contrato.

## 11. Tratamento de erros

Garantir os seguintes erros, com corpo JSON no formato `{"erro":"codigo"}`:

| HTTP | Código |
|---|---|
| 422 | `placa_invalida` |
| 422 | `entrada_invalida` |
| 422 | `data_invalida` |
| 404 | `bilhete_nao_encontrado` |
| 409 | `bilhete_ja_encerrado` |
| 409 | `bilhete_nao_aberto` |
| 409 | `bilhete_em_aberto` |

**Critério de conclusão:** todos os erros definidos retornam exatamente os códigos e corpos JSON previstos.

## 12. Testes automatizados — TDD

Utilizar pytest e os cenários definidos em `tests.md`.

Para cada funcionalidade:
1. Preparar o teste do comportamento esperado.
2. Implementar o comportamento necessário.
3. Executar os testes.
4. Corrigir falhas e refatorar sem alterar o contrato.

Cobrir especialmente:
- Abertura e validação de placa.
- Entrada opcional e seu fuso.
- Fração exata e minuto adicional.
- Tolerância zero e teto.
- Estados e conflitos de placa.
- Cancelamento e encerramento.
- Listagens e ordenação.
- Relatório diário e média arredondada.
- Erros HTTP e campos obrigatórios.

**Critério de conclusão:** testes automatizados verificam os casos de uso e os limites definidos no contrato.

## 13. Container e documentação

- Criar `Containerfile`.
- Configurar o serviço para escutar na porta `8005`.
- Garantir a instalação das dependências necessárias.
- Criar README com instruções de instalação, execução local, testes e execução em container.

**Critério de conclusão:** a aplicação gerada possui os artefatos necessários para instalação, inicialização e execução dos testes.

## 14. Verificação final

- Conferir os oito casos de uso do `spec.md`.
- Validar os cenários de `tests.md`.
- Verificar os parâmetros da variante.
- Confirmar os campos e códigos HTTP do contrato.
- Executar os testes automatizados.
- Conferir a documentação e a execução pela porta `8005`.

**Critério de conclusão:** implementação compatível com o contrato oficial, sem requisitos adicionais não autorizados.