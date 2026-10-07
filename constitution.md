Constitution — Zona Azul Digital

1. Objetivo

O sistema deve fornecer exclusivamente uma API REST para gerenciamento de bilhetes de estacionamento rotativo.

A API deve permitir:

abrir bilhetes;

encerrar bilhetes;

cancelar bilhetes;

listar bilhetes ativos;

consultar histórico por placa;

gerar relatório diário.

Não deve ser criado back-office ou interface gráfica.

2. Variante obrigatória

A implementação deve utilizar obrigatoriamente:

TARIFA_HORA_CENTAVOS = 600

FRACAO_MINUTOS = 30

TETO_DIARIO_CENTAVOS = 5000

TOLERANCIA_MINUTOS = 0

PORTA_SERVICO = 8005

O serviço deve escutar na porta 8005.

3. Convenções gerais

A API deve utilizar JSON.

Valores monetários devem ser sempre representados como números inteiros em centavos.

Valores monetários nunca devem ser retornados como ponto flutuante.

Datas e horários devem utilizar ISO-8601 conforme o contrato.

A entrada dos bilhetes deve possuir fuso -03:00.

IDs devem identificar unicamente cada bilhete.

O contrato REST possui prioridade sobre exemplos ilustrativos.

O exemplo antigo com campo valor em ponto flutuante deve ser ignorado.

4. Estados de um bilhete

Um bilhete pode possuir somente um dos seguintes status:

aberto;

encerrado;

cancelado.

Somente bilhetes com status aberto podem ser encerrados ou cancelados.

5. Placa

A placa deve possuir:

exatamente 7 caracteres;

somente caracteres alfanuméricos;

somente letras maiúsculas quando houver letras.

Placa ausente ou inválida deve retornar:

HTTP 422

{"erro":"placa_invalida"}

6. Regra de abertura

Uma placa pode possuir no máximo um bilhete aberto simultaneamente.

Caso já exista um bilhete aberto para a placa:

HTTP 409

{"erro":"bilhete_em_aberto"}

Depois que o bilhete for encerrado ou cancelado, a placa pode abrir um novo bilhete.

7. Regra de cobrança

Cada fração possui 30 minutos.

Uma hora possui duas frações.

Como a tarifa por hora é 600 centavos:

uma fração de 30 minutos custa 300 centavos.

O número de frações deve sempre ser arredondado para cima.

Exemplos:

1 minuto → 1 fração;

30 minutos → 1 fração;

31 minutos → 2 frações;

60 minutos → 2 frações;

61 minutos → 3 frações.

8. Tolerância

A variante possui:

TOLERANCIA_MINUTOS = 0

Portanto, qualquer duração positiva deve gerar cobrança.

A lógica deve respeitar a regra geral:

duração menor ou igual à tolerância → valor 0;

ultrapassou a tolerância → cobrar desde o primeiro minuto.

A tolerância não deve ser descontada da duração cobrada.

9. Teto diário

O valor de um bilhete nunca pode ultrapassar:

5000 centavos.

[!WARNING]
O teto deve ser aplicado após o cálculo normal pelas frações.

10. Cancelamento

Um bilhete cancelado:

recebe status cancelado;

não gera cobrança;

não gera saida;

não gera valor_centavos.

11. Encerramento de bilhete cancelado

O contrato determina que somente bilhetes abertos podem ser encerrados, mas não especifica explicitamente qual erro deve ser retornado ao tentar encerrar um bilhete cancelado.

Por esse motivo, nenhum código HTTP adicional foi definido para essa situação. O comportamento deve ser confirmado no enunciado oficial antes de ser especificado.

12. Escopo do fuso horário

O contrato exige que o campo opcional entrada utilize ISO-8601 com fuso horário.

O exemplo de resposta utiliza o fuso -03:00, mas isso não é suficiente para concluir que todos os campos temporais da aplicação devam obrigatoriamente utilizar esse mesmo fuso.

A especificação deve preservar a exigência de ISO-8601 com fuso sem ampliar a regra além do que o contrato determina.

13. Ordenação

As seguintes consultas devem apresentar os bilhetes mais recentes primeiro:

listagem de bilhetes ativos;

histórico por placa.

14. Erros obrigatórios

Situação |HTTP Erro
Placa ausente ou inválida| 422 |placa_invalida
Entrada inválida| 422 |entrada_invalida
Data inválida| 422| data_invalida
Bilhete inexistente| 404 |bilhete_nao_encontrado
Bilhete já encerrado |409 |bilhete_ja_encerrado
Cancelar bilhete não aberto| 409 |bilhete_nao_aberto
Placa já possui bilhete aberto |409 |bilhete_em_aberto