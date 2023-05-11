
# PowerHUB API

Esta é uma API construída com a linguagem Python usando o framework FastAPI. Ela foi desenvolvida para fornecer uma solução de gerenciamento de relatórios para organizações, com recursos como:

- Autenticação de usuários com token JWT
- Gerenciamento de usuários, relatórios e organizações
- Integração com Stripe para processamento de pagamentos **Em implementaçaõ**
- Documentação detalhada da API com Swagger
- Integração com Azure e PowerBI **Em desenvolvimento**

Esta API é fácil de usar e altamente escalável, permitindo que sua organização gerencie seus relatórios de maneira eficiente e segura. Sinta-se à vontade para explorar os recursos e experimentar a API em sua própria plataforma.

## Funcionalidades

- Gestão de Usuário
- Gestão de Grupos
- Relatório
- Multiplas Empresas


## Documentação da API

#### Autenticação

```http
  POST /login
```

| Parâmetro   | Tipo        | Descrição                           |
| :---------- | :---------  | :---------------------------------- |
| `email`     | `string`    |  **Obrigatório**. Login do usuário  |
| `password`  | `string`    |  **Obrigatório**. password          |


A rota acima retorna o id_user, id_company e o token de autenticação.


```bash
@router.post('/login', response_model=UserLoginResponse, tags=['Authentication'])
def login(user_login_request: UserLoginRequest = Body(...)):
    with SessionLocal() as session:
        email = user_login_request.email
        password = user_login_request.password
        user = session.query(User).filter(User.DS_EMAIL == email).first()

        if user:
            password_criptografada = hashlib.sha256(password.encode()).hexdigest()
            if password_criptografada == user.DS_SENHA:
                user_info = {
                    'id_user': user.ID_USUARIO,
                    'nome_user': user.NOME_USUARIO,
                    'email_user': user.DS_EMAIL,
                    'id_company': user.ID_ORGANIZACAO,
                    'administrator': user.FL_ADMINISTRADOR
                }
                token = create_access_token(user_info)
                response = UserLoginResponse(success=True, message='Autenticado com sucesso', token=token, user_info=user_info)
                return response
    raise HTTPException(status_code=401, detail='Senha incorreta ou Usuário não encontrado!')
```

### Listar Grupos 

A rota listar grupos seria a possível home da aplicação. Esse endpoint exige obrigatoriamente que o token gerado na rota de `login`. 

| Parâmetro         | Tipo        | Descrição                           |
| :----------       | :---------  | :---------------------------------- |
| `id_usuario`      | `int`       |  **Obrigatório**. id_usuario        |
| `password`        | `string`    |  **Obrigatório**. token             |

O recurso de acesso por token foi implementado com a biblioteca PyJWT. Abaixo a função de decodificação do token utiliza. 

```bash
def get_auth_user(token: str = Header(..., description='Authorization token', example='Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...',), id_usuario: int = None):
    try:
        token = token.split(" ")[1]
        payload = jwt.decode(token, JWT_SECRET_KEY, algorithms=["HS256"])
        user_id = payload.get("id_user")
        if datetime.datetime.utcnow() > datetime.datetime.fromtimestamp(payload['exp']):
            raise HTTPException(status_code=401, detail="O token fornecido expirou.")
        if id_usuario is not None and id_usuario != user_id:
            raise HTTPException(status_code=401, detail="O ID do usuário no token não corresponde ao ID do usuário fornecido.")
        return user_id
    except:
        raise HTTPException(status_code=401, detail="O token fornecido é inválido ou expirou. Verifique se o token está correto ou gere um novo.")
```

### Configurações de conexão

Crie e edite um arquivo .env com as informações de conexão com o banco de dados e a chave secreta para validar a assinatura do token. É recomendado utilizar o comando openssl para gerar uma chave secreta, que pode ser obtida através do comando `openssl rand -hex 32`.

No arquivo .env, você precisa definir as seguintes variáveis:

`JWT_SECRET_KEY`: uma string aleatória que será utilizada como chave secreta para a assinatura do token JWT.
`JWT_ALGORITHM`: algoritmo de assinatura que será utilizado. Recomenda-se utilizar o valor HS256.
`JWT_EXPIRATION_TIME`: Tempo de expiração do token JWT, em minutos.

Além disso, é necessário definir a variável `DATABASE_URL` com a URL de conexão com o banco de dados que você está utilizando. No exemplo dado, a URL de conexão é `mysql+pymysql://user:password@localhost/dbname`.



## Licença

[MIT](https://choosealicense.com/licenses/mit/)

