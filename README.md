# 📊 Dashboard de Vendas e Atendimento via WhatsApp

Dashboard interativo em **Streamlit** para acompanhamento e análise de vendas e atendimentos de uma equipe de vendas pelo WhatsApp, com meta x realizado, comparação entre períodos e auditoria de cada número exibido.

> 🔒 **O código deste projeto é privado**, pois foi desenvolvido para uso interno de uma empresa. Este repositório existe apenas para apresentar o projeto: o problema, a solução, a arquitetura e as decisões técnicas.

---

## 🎯 O problema

Os dados de vendas e atendimento vinham de sistemas diferentes, exportados em CSV, e eram consolidados manualmente. Isso tornava difícil acompanhar a meta do dia, comparar períodos e, principalmente, **confiar nos números**: ninguém conseguia ver de onde cada valor tinha saído.

## ✅ A solução

Um pipeline em duas etapas independentes e um dashboard que trabalha só com bases já tratadas:

1. **Ingestão:** scripts leem os CSVs exportados de cada sistema, tratam, cruzam e gravam bases padronizadas em `.parquet`.
2. **Dashboard:** lê apenas os `.parquet`, calcula as métricas e exibe tudo de forma interativa.

Separar as duas etapas deixa o dashboard rápido e simples, e permite reprocessar uma base sem mexer nas outras.

## ✨ Funcionalidades

- **Meta x realizado:** meta diária, GAP e percentual de atingimento, com a meta do mês calculada pela soma das diárias.
- **Comparação entre períodos:** variações (▲ / ▼) contra um período equivalente anterior: mesmo dia da semana na semana anterior, mesmo intervalo do mês anterior e mesmos dias da semana 4 semanas antes. Se não há base no período anterior, a variação não aparece, em vez de mostrar um número falso.
- **Venda por origem do atendimento:** atribuição das vendas a botões e rastreadores, campanhas, orgânico e anúncios, cruzando pedidos e atendimentos pelo telefone, de modo que a soma das origens fecha com a venda total.
- **Fluxo de atendimentos:** mapa de calor por dia da semana e hora.
- **Desempenho por vendedora:** atendimentos, pedidos, faturamento, conversão e ticket médio no dia selecionado.
- **Auditoria:** todo número agregado pode ser aberto nos registros que o compõem, e a tabela dia a dia usa exatamente o mesmo cálculo dos cards, então os totais sempre batem.
- **Controle de acesso:** login com perfis (leitor, admin, superadmin), senhas guardadas apenas como hash (PBKDF2-SHA256 com salt) e registro de logins.
- **Administração:** tela com situação de cada base, processamento dos arquivos novos pelo próprio dashboard e checagens automáticas de saúde dos dados (bases desatualizadas, dias sem dados, dias sem meta, inconsistências de origem).

## 🧠 Decisões técnicas

- **Ingestão idempotente:** reprocessar um período já carregado não duplica registros. A versão nova substitui a antiga pela chave do registro, o que permite reexportar dados para atualizar status.
- **Segurança na ingestão:** os arquivos só são movidos para a pasta de processados se a base for gravada com sucesso, então um erro nunca perde dados.
- **Fonte única dos números:** todas as métricas são calculadas em um único módulo e reaproveitadas por cards, tabelas e auditoria, o que evita divergência entre telas.
- **Atualização automática:** o dashboard percebe quando uma base muda (pela data de modificação do arquivo) e mostra os dados novos sem reiniciar.
- **Regras de negócio explícitas:** definições como pedido-base (pedidos divididos contam uma vez) e exclusão de cancelados estão documentadas e aplicadas de forma consistente.

## 🛠️ Stack

![Python](https://img.shields.io/badge/-Python-3776AB?style=flat-square&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/-Pandas-150458?style=flat-square&logo=pandas&logoColor=white)
![Streamlit](https://img.shields.io/badge/-Streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white)
![Plotly](https://img.shields.io/badge/-Plotly-3F4F75?style=flat-square&logo=plotly&logoColor=white)

Além de **pyarrow** (leitura e escrita de `.parquet`) e **Altair** (gráficos).

## 🗂️ Arquitetura

```text
CSVs exportados dos sistemas
        │
        ▼
  Ingestão (scripts por base)
  tratamento · cruzamento · deduplicação
        │
        ▼
  Bases padronizadas (.parquet)
        │
        ▼
  Dashboard (Streamlit)
  métricas · auditoria · login · administração
```

## 🗺️ Próximos passos

- Edição de metas e tabelas de-para pela tela de administração
- Manutenção de sessão ao recarregar a página e limite de tentativas de login
- Unificação da lógica de ingestão em funções reutilizáveis
- Testes automáticos das regras de ingestão e da auditoria

## 👤 Meu papel

Desenvolvi o projeto de ponta a ponta: levantamento das regras de negócio, ingestão e tratamento dos dados, cálculo das métricas, interface do dashboard, autenticação e administração.

Hoje graças a ele consigo trazer os resultados do canal 85% mais rápido em comparação com a atualização manual do dashboard antigo, garantindo que o canal ganhe tempo para planejar o próximo passo para alcançar a meta.
