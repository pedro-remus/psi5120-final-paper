# PSI5120 — Trabalho Final: Extensão do pipeline serverless da Aula 07

Trabalho final da disciplina PSI5120 (Tópicos em Computação em Nuvem). Estende o laboratório serverless da Aula 07 (Lambda + SQS + DLQ + CloudFormation) com API Gateway, DynamoDB (persistência idempotente), observabilidade (CloudWatch + SNS) e uma avaliação experimental de latência/throughput/confiabilidade sob carga.

## Links rápidos

- **O artigo IEEE está em** [`paper/main.tex`](paper/main.tex). Para compilar: crie um projeto novo no [Overleaf](https://www.overleaf.com) (New Project → Upload Project, ou cole o conteúdo em um projeto "Blank"), enviando `paper/main.tex` junto com as pastas `results/figures/` e `evidencias/screenshots/` (as figuras são referenciadas por caminho relativo a partir de `paper/`). A classe `IEEEtran` já vem pronta no Overleaf, não precisa instalar nada.
- **Quer ver as evidências da execução?** Veja [`evidencias/README.md`](evidencias/README.md) — checklist com o que cada arquivo comprova (identidade AWS, deploy, testes manuais, teste de carga, dashboard, e-mail de alarme).
- **Quer rodar o projeto na sua própria conta AWS?** Use os scripts em `scripts/` (`deploy.sh`, `cleanup.sh`, `load_test.py`, `analyze_results.py`) — veja a seção "Início rápido" abaixo.

## Arquitetura (resumo)

```
Cliente → API Gateway (HTTP API) → Lambda "producer" → SQS (fila principal)
                                                            │
                                        (falha após 2 tentativas)
                                                            │
                                                            ▼
                                        Lambda "consumer" → DynamoDB (escrita idempotente)
                                                            │
                                           (em caso de falha) → DLQ → Alarme CloudWatch → SNS (e-mail)
```

Detalhes completos no artigo (`paper/main.tex`, seção System Design).

## Estrutura do repositório

```
.
├── evidencias/
│   ├── README.md                # checklist de evidências mínimas
│   ├── screenshots/              # capturas de tela do Console AWS
│   ├── cli-output/                # saídas de comandos salvas em .txt
│   └── logs/                      # exportações de CloudWatch Logs
├── infra/
│   └── template.yaml           # CloudFormation: toda a infraestrutura da stack
├── src/
│   ├── producer_function.py    # Lambda atrás do API Gateway (cópia legível do código embutido no template)
│   └── consumer_function.py    # Lambda disparada pela fila SQS (idem)
├── scripts/
│   ├── deploy.sh                # deploy da stack via AWS CLI
│   ├── cleanup.sh               # remoção da stack + checagem de resíduos
│   ├── load_test.py             # gera carga e mede latência/sucesso
│   └── analyze_results.py       # gera os gráficos usados no artigo
├── results/
│   ├── raw/                     # CSVs brutos do teste de carga
│   └── figures/                 # gráficos gerados a partir dos CSVs
└── paper/
    └── main.tex                 # artigo em formato IEEE (IEEEtran)
```

## Origem do código

Este projeto parte do laboratório didático da Aula 07 (`psi5120_a07_lambda_function.py` e `template_cloudformation.yaml`, disponíveis no material da disciplina), que demonstra o mecanismo básico de retry/DLQ com uma única função Lambda. A lógica de normalização de evento SQS/direto e a demonstração de falha controlada foram mantidas como estão nesse laboratório; as extensões (API Gateway, DynamoDB, observabilidade) estão descritas no artigo, Seção IV (System Design).

## Início rápido

```bash
chmod +x scripts/deploy.sh scripts/cleanup.sh
./scripts/deploy.sh SEU_EMAIL@exemplo.com psi5120-tf-serverless us-east-1 SEU_PROFILE_AWS
```

Depois do deploy, use `scripts/load_test.py` para gerar carga e `scripts/analyze_results.py` para gerar os gráficos; rode `scripts/cleanup.sh` ao final para remover os recursos da AWS.

## Autoria

- Pedro Remus de Ávila (NUSP 13682486)
- Murilo Gabriel Moraes de Azevedo (NUSP 13782776)
- Bruno Valle Martins (NUSP 13681036)

Disciplina PSI5120 — Tópicos em Computação em Nuvem, Escola Politécnica da Universidade de São Paulo, 2026.
