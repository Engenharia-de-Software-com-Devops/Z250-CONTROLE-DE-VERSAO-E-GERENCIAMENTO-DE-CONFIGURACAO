# Lab 3.1 — Git Internals com Plumbing Commands: Lições Aprendidas

Laboratório com 10 exercícios explorando o Git por dentro: objetos, index, referências, tags, alcançabilidade e construção de histórico completo sem `git add` / `git commit`.

## Modelo mental

```
HEAD ──► refs/heads/main ──► commit ──► tree ──► blob (conteúdo)
                               │          └────► tree (subdiretório)
                               └─► parent (commit anterior)

refs/tags/v1.0.0 ──► objeto tag ──► commit      (annotated)
refs/tags/v1-light ─────────────► commit       (lightweight)
```

- **Objetos** (blob, tree, commit, tag) são imutáveis e endereçados pelo SHA-1 do conteúdo.
- **Referências** (branches, tags, HEAD) são ponteiros mutáveis para objetos.
- **Index** é o rascunho do próximo snapshot: lista de `modo + hash + caminho`.

## Lições por exercício

### 1. Estrutura do `.git`
- `.git/objects` é o banco de objetos; pasta = 2 primeiros caracteres do hash, arquivo = 38 restantes.
- `.git/HEAD` normalmente contém `ref: refs/heads/main`, não um hash. `git rev-parse HEAD` resolve a cadeia HEAD → branch → commit.
- Um repositório com um único commit já tem 3 objetos: blob, tree e commit.

### 2. Blob
- `git hash-object` só calcula o hash; com `-w` grava o objeto.
- Blob guarda apenas `blob <tamanho>\0<conteúdo>` — **não** guarda o nome do arquivo.
- `echo` adiciona `\n`: `"Git armazena conteudo"` (21 chars) gerou blob de 22 bytes.

### 3. Deduplicação
- Arquivos com nomes diferentes e conteúdo igual geram o **mesmo hash** e compartilham um único blob.
- Gravar duas vezes com `-w` não duplica nada: o endereço é o próprio conteúdo.
- Alterar um byte gera hash totalmente diferente.

### 4. Index via `update-index`
- `git update-index --add --cacheinfo 100644,<hash>,<caminho>` faz o papel do `git add` sem ler o working tree.
- Modos: `100644` arquivo comum, `100755` executável, `120000` symlink, `040000` tree, `160000` submódulo.
- Estar no index ≠ estar em um commit. O blob existe no banco; o index só aponta para ele.

### 5. Tree
- `git write-tree` cria a tree a partir do index, sem precisar de commit.
- Tree não guarda conteúdo, só entradas `modo tipo hash nome`. Subdiretório = tree dentro de tree.
- `ls-tree` mostra um nível; `ls-tree -r` desce recursivamente e lista os blobs com caminho completo.

### 6. Commit via `commit-tree`
- Commit = tree + parent(s) + author + committer + mensagem. Mudar qualquer campo muda o hash.
- `-p` define o pai; sem ele o commit vira raiz.
- Criar o objeto commit **não move nenhuma branch**.
- Vários commits podem apontar para a mesma tree.

### 7. Referências
- `git update-ref refs/heads/experimento <hash>` cria uma branch: é só um arquivo com um hash.
- Branch é ponteiro móvel; commit é objeto imutável.
- `git update-ref <ref> <novo> <antigo>` funciona como compare-and-swap: só atualiza se a ref ainda estiver no valor antigo.
- Remover a ref não apaga os commits; eles ficam inacessíveis até o `gc`.

### 8. Tags
- Lightweight: apenas ref → `git cat-file -t` retorna `commit`.
- Annotated: objeto próprio (`object`, `type`, `tag`, `tagger`, mensagem) → `cat-file -t` retorna `tag`.
- `git rev-parse v1.0.0` devolve o hash do objeto tag; `git rev-parse 'v1.0.0^{}'` "descasca" até o commit.

### 9. Objetos inacessíveis
- `git fsck --full --no-reflogs --unreachable` lista objetos válidos que nenhuma raiz (refs, HEAD, index) alcança.
- No lab ficaram inacessíveis: blobs criados só com `hash-object -w` e a primeira tree que nunca virou commit.
- Inacessível ≠ inválido. O `git gc` remove apenas após `gc.pruneExpire` (padrão 2 semanas) — margem para recuperação.
- `git count-objects -v` confirma a contagem: cada passo do lab somou objetos previsíveis.

### 10. Histórico só com plumbing
Sequência equivalente a `git add` + `git commit`:

```bash
BLOB=$(git hash-object -w app.txt)
git update-index --add --cacheinfo 100644,"$BLOB",app.txt
TREE=$(git write-tree)
COMMIT=$(echo "msg" | git commit-tree "$TREE" -p "$PARENT")
git update-ref refs/heads/main "$COMMIT" "$PARENT"
```

Tag anotada com `git mktag` (valida o formato antes de gravar) + `git update-ref refs/tags/v1.0.0 "$TAG_OBJECT"`.

Resultado validado: 7 objetos (2 blobs, 2 trees, 2 commits, 1 tag), `main` no segundo commit, tag apontando para ele, `git fsck --full` sem erros, `git status` limpo.

## Armadilhas encontradas

| Sintoma | Causa | Correção |
|---|---|---|
| `find: Argumento desconhecido para -type: -` | `-type -f` | `-type f` |
| `error:  --cacheinfo: does not exist` | `\ ` (barra + espaço) escapa o espaço e o argumento vira `" --cacheinfo"` | barra só no fim da linha, sem nada depois |
| `git log` sem saída | `less` sem `-X` limpa a tela ao sair (`LESS=-R`) | `git --no-pager log ...` ou `export LESS=-RFX` |
| Prompt mostra `✗` antes do primeiro `update-ref` | branch `main` ainda não existia; index difere de "nada" | some após `update-ref refs/heads/main` |

## Conclusão

Os comandos Porcelain são combinações dos Plumbing. Entender a separação entre **objetos imutáveis** e **referências mutáveis** explica branches, tags, recuperação de commits "perdidos" e o funcionamento do garbage collection.
