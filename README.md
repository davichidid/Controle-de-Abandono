# Controle de Abandono e Eficiência Operacional
  
Esta query tem como objetivo mensurar a assertividade e eficácia do fluxo de atendimento ao cliente, visando reduzir o abandono de interações que geram custos excedentes de telefonia ou canais digitais sem a efetivação do contato com o cliente ou atendente.
Através deste script, é possível extrair uma síntese consolidada da representatividade do abandono em relação aos atendimentos de êxito, permitindo tanto uma análise em tempo real quanto uma visão retroativa, estabelecendo uma média e fazendo um comparativo com outro período do mesmo nicho.



O código atua como a extração de dados brutos (Raw Data) que alimenta uma estrutura de Power Query + VBA. Esse ecossistema permite:

* **Segmentação Dinâmica:** Análise detalhada por segmento extratificando a visão de acordo com os blocos de atendimento, ajudando a identificar de maneira mais rápida a origem do desvio.

* **Identificação de Causa Raiz:** Rapidez na detecção de picos de abandono e entendimento dos motivos (falhas técnicas ou dimensionamento).

* **Atualização Automatizada:** O relatório final é atualizado a cada 30 minutos, garantindo suporte à decisão no intraday.

* **Multicanalidade:** Monitoramento aplicável a discadores, fluxos receptivos (0800), Agentes Virtuais e ferramentas de negociação.


```
SELECT
    DATEPART(HOUR, [VW_FLUXO_LIGACOES_REALTIME].[INICIO_ATENDIMENTO]) AS [HORA_ATENDIMENTO],

    -- Total de Chamadas Conectadas (Sucesso de atendimento)
    COUNT(
        CASE 
            WHEN [VW_FLUXO_LIGACOES_REALTIME].[STATUS_LIGACAO] IN (
                'ATENDIMENTO_AGENTE',
                'ATENDIMENTO_CLIENTE',
                'ATENDIMENTO_SISTEMA'
            )
            THEN 1
        END
    ) AS [TOTAL_ATENDIMENTOS],

    -- Total de Chamadas de Abandono (Perdidas em fila)
    COUNT(
        CASE 
            WHEN [VW_FLUXO_LIGACOES_REALTIME].[STATUS_LIGACAO] IN (
                'ABANDONO_FILA',
                'TEMPO_MAXIMO_FILA',
                'SEM_AGENTES_DISPONIVEIS',
                'CHAMADA_CANCELADA',
                'SOLICITACAO_CALLBACK'
            )
            THEN 1
        END
    ) AS [ABANDONO]

FROM
    [DB_OPERACIONAL].[SCHEMA_PLANNING].[VW_FLUXO_LIGACOES_REALTIME] WITH (NOLOCK)

WHERE
    [VW_FLUXO_LIGACOES_REALTIME].[CARTEIRA_ID] LIKE '%PROJETO_ESTRATEGICO%' 
    AND CONVERT(VARCHAR(8), [VW_FLUXO_LIGACOES_REALTIME].[INICIO_ATENDIMENTO], 108) >= '08:00:00'

GROUP BY
    DATEPART(HOUR, [VW_FLUXO_LIGACOES_REALTIME].[INICIO_ATENDIMENTO])

ORDER BY
    [HORA_ATENDIMENTO];
```
