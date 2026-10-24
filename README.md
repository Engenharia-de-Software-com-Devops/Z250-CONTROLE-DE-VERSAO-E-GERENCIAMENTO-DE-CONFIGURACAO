# Z250 — Controle de Versão e Gerenciamento de Configuração

Repositório de registro dos laboratórios da disciplina **Z250 — Controle de Versão e Gerenciamento de Configuração**, da pós-graduação em Engenharia de Software com DevOps.

Cada laboratório tem sua própria pasta em `labs/`, com as lições aprendidas, os comandos usados e os problemas encontrados durante a execução.

## Laboratórios

| Lab | Tema | Conteúdo | Nível |
|---|---|---|---|
| [3.1](labs/lab3_1-git-plumbing/LICOES-APRENDIDAS.md) | Git Internals — Plumbing Commands | Objetos (blob, tree, commit, tag), index, referências, objetos inacessíveis e construção de histórico sem `git add`/`git commit` | Básico → Avançado |

## Estrutura do repositório

```
.
├── README.md                         # este índice
└── labs/
    └── lab3_1-git-plumbing/
        └── LICOES-APRENDIDAS.md      # lições, comandos e armadilhas do lab
```

## Padrão de documentação

Novos laboratórios seguem a mesma organização:

1. Pasta `labs/labX_Y-<tema>/` (nome em minúsculas, separado por hífens).
2. Arquivo `LICOES-APRENDIDAS.md` contendo:
   - **Modelo mental** — diagrama ou resumo do conceito central.
   - **Lições por exercício** — o que cada comando revelou.
   - **Armadilhas encontradas** — tabela `sintoma | causa | correção`.
   - **Conclusão** — síntese em poucas linhas.
3. Nova linha na tabela **Laboratórios** deste README.

## Convenções

- Mensagens de commit no padrão [Conventional Commits](https://www.conventionalcommits.org/) (`docs:`, `feat:`, `fix:`...).
- Comandos e saídas em blocos de código; hashes abreviados para 7 caracteres.
- Experimentos são executados em repositórios descartáveis, nunca em projetos reais.

## Referências

- [Pro Git — Capítulo 10: Git Internals](https://git-scm.com/book/pt-br/v2/Git-Internals-Plumbing-and-Porcelain)
- [Documentação oficial do Git](https://git-scm.com/docs)
