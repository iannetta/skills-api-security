# Hands-on: Segurança em APIs (45 minutos)

Hands-on prático usando **Google Colab + FastAPI + Swagger UI**. Todo mundo acompanha sem instalar nada localmente, e dá para demonstrar cada conceito visualmente, direto no navegador.

> "O atacante normalmente não vê sua interface. Ele vê sua API."

## Objetivo

Ao final da sessão, os participantes serão capazes de:

- Identificar vulnerabilidades comuns em APIs
- Entender como elas são exploradas
- Aplicar correções simples
- Relacionar os problemas ao [OWASP API Top 10](https://owasp.org/www-project-api-security/)

## Agenda (45 min)

| Parte | Tema | Tempo |
|---|---|---|
| 1 | Introdução | 5 min |
| 2 | Ambiente | 5 min |
| 3 | Vulnerabilidade 1 — BOLA | 10 min |
| 4 | Vulnerabilidade 2 — Excessive Data Exposure | 10 min |
| 5 | Vulnerabilidade 3 — Falta de Rate Limiting | 10 min |
| 6 | Encerramento | 5 min |

---

## Parte 1 — Introdução (5 min)

Apresentar rapidamente:

- O que é uma API
- O que é o OWASP API Top 10
- Por que APIs são alvo de ataque

**Mensagem central da sessão:**

> O atacante normalmente não vê sua interface. Ele vê sua API.

---

## Parte 2 — Ambiente (5 min)

Tudo roda no Google Colab, sem instalação local.

### 1. Instalar dependências

```bash
pip install fastapi uvicorn pyngrok
```

### 2. Subir uma API vulnerável

Cole o código base abaixo em uma célula do Colab (as vulnerabilidades específicas de cada parte serão adicionadas a este arquivo ao longo do hands-on).

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"status": "API no ar"}
```

### 3. Expor com ngrok

```python
from pyngrok import ngrok
import nest_asyncio
import uvicorn

nest_asyncio.apply()

public_url = ngrok.connect(8000)
print("URL pública:", public_url)

uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 4. Acessar o Swagger UI

Abra a URL pública gerada pelo ngrok e acrescente `/docs` (Swagger UI do FastAPI) para explorar a documentação interativa:

```
https://<sua-url-ngrok>.ngrok-free.app/docs
```

Mostre para a turma que **todos os endpoints estão documentados e disponíveis publicamente** — inclusive os que não deveriam ser expostos sem autenticação.

---

## Parte 3 — Vulnerabilidade 1: BOLA (10 min)

**BOLA = Broken Object Level Authorization** (OWASP API1).

### Código vulnerável

```python
orders = {
    100: {"owner": "iannetta", "value": 500},
    101: {"owner": "luigi", "value": 800}
}

@app.get("/orders/{id}")
def get_order(id: int):
    return orders[id]
```

### Demonstração

No Swagger, execute:

```
GET /orders/100
GET /orders/101
```

**Pergunta para a turma:** o que impede um usuário de acessar pedidos de outra pessoa?

**Resposta:** nada. Basta trocar o número na URL.

### Correção

```python
@app.get("/orders/{id}")
def get_order(id: int, current_user="luis"):
    order = orders[id]
    if order["owner"] != current_user:
        raise HTTPException(status_code=403)
    return order
```

A correção compara o dono do recurso com o usuário autenticado antes de devolver os dados.

---

## Parte 4 — Vulnerabilidade 2: Excessive Data Exposure (10 min)

Relacionada ao OWASP **API3 — Broken Object Property Level Authorization / Excessive Data Exposure**.

### Código vulnerável

```python
users = {
    1: {
        "name": "Luigi",
        "email": "iannetta@email.com",
        "cpf": "12345678900",
        "salary": 15000,
        "internalScore": 900
    }
}

@app.get("/profile")
def profile():
    return users[1]
```

### Pergunta

O frontend precisa de tudo isso? **Normalmente não.**

### Correção

```python
@app.get("/profile")
def profile():
    return {
        "name": users[1]["name"]
    }
```

O endpoint deve devolver apenas os campos que a tela realmente consome — nunca o objeto interno completo.

---

## Parte 5 — Vulnerabilidade 3: Falta de Rate Limiting (10 min)

Relacionada ao OWASP **API4 — Unrestricted Resource Consumption**.

### Endpoint

```python
@app.post("/login")
def login():
    return {"message": "trying"}
```

### Demonstração

Execute várias chamadas seguidas contra `/login` e mostre que nada bloqueia o excesso de requisições.

Explique os riscos:

- **Brute force**
- **Credential stuffing**
- **Enumeração de usuários**

### Mitigação

- Rate limiting
- Lockout de conta
- CAPTCHA
- MFA

---

## Parte 6 — Encerramento (5 min)

Relacione cada vulnerabilidade demonstrada com o OWASP API Top 10:

| Vulnerabilidade | OWASP API Top 10 |
|---|---|
| BOLA | **API1** — Broken Object Level Authorization |
| Excessive Data Exposure | **API3** — Broken Object Property Level Authorization / Excessive Data Exposure |
| Falta de Rate Limiting | **API4** — Unrestricted Resource Consumption |

---

## Material de apoio para os Champions

Após o hands-on, compartilhe o cheat sheet abaixo com o time.

### Cheat Sheet — revisão de APIs antes do merge

- [ ] O endpoint exige autenticação?
- [ ] Existe autorização checando o dono do recurso?
- [ ] O retorno contém apenas os dados necessários?
- [ ] Há proteção contra abuso (rate limiting)?
- [ ] Os inputs são validados?

---

## Referências

- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Documentação do FastAPI](https://fastapi.tiangolo.com/)
- [pyngrok](https://pyngrok.readthedocs.io/)

