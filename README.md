# SECRE·TINA

Assistente financeira pessoal (PWA), hospedada em secretina.barukstore.com.br.

## Stack

- HTML/JS estático, sem bundler
- Firebase Auth + Firestore (projeto `secretina-1d89a`)
- GitHub Pages (domínio próprio via `CNAME`)

## Desenvolvimento

```
python3 -m http.server 8080
```

## Deploy

Push para `main` publica automaticamente em secretina.barukstore.com.br
(GitHub Pages serve a raiz da branch, sem build).

```
git push
```
