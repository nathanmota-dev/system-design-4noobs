# Autenticação e autorização

Autenticação é o processo de verificar quem o usuário é. Autorização é o processo de verificar se esse usuário tem permissão para executar uma ação ou acessar um recurso.

## Autenticação

A autenticação normalmente utiliza um ou mais fatores:

- **Conhecimento:** algo que a pessoa sabe, como uma senha.
- **Posse:** algo que a pessoa possui, como um celular, token ou chave física.
- **Inerência:** algo que faz parte da pessoa, como biometria facial ou impressão digital.

Quando o sistema combina fatores de categorias diferentes, temos autenticação multifator, como 2FA ou MFA.

### Sessões e tokens

Em uma sessão, o servidor mantém o estado de autenticação em um armazenamento, como Redis, e o cliente recebe apenas um identificador de sessão com tempo de expiração.

Um token carrega informações que permitem validar o acesso. Um JWT, por exemplo, é um formato de token assinado que pode ser validado sem consultar uma sessão a cada requisição. O trade-off é que a revogação antes da expiração exige uma estratégia adicional, como tokens curtos, rotação ou uma lista de revogação.

### Tecnologias e padrões comuns

- **Amazon Cognito, Keycloak e Auth0:** provedores ou plataformas de identidade.
- **OAuth 2.0:** protocolo de autorização delegada. Ele permite que uma aplicação acesse recursos em nome do usuário sem receber sua senha.
- **OpenID Connect (OIDC):** camada de identidade construída sobre OAuth 2.0, usada para autenticação e *single sign-on*.
- **JWT:** formato de token; não é, por si só, um serviço nem um protocolo completo de autenticação.
- **Passkey:** credencial baseada em criptografia de chave pública, normalmente desbloqueada no dispositivo por biometria ou PIN.
- **2FA/MFA:** uso de dois ou mais fatores de autenticação.

## Autorização

Autorização verifica se o usuário autenticado pode acessar um recurso. Alguns modelos comuns são:

- **RBAC — Role-Based Access Control:** permissões são associadas a papéis, e os usuários recebem esses papéis.
- **ABAC — Attribute-Based Access Control:** a decisão usa atributos do usuário, do recurso e do contexto.
- **ACL — Access Control List:** cada recurso mantém uma lista de identidades e permissões.
- **Permissões granulares:** ações específicas, como `order:read` e `order:cancel`, são avaliadas individualmente.

## System Design com autenticação e autorização

No fluxo abaixo, uma camada de borda, como API Gateway ou proxy reverso, pode validar o token antes de encaminhar a requisição. A autorização de domínio, porém, também deve ser verificada pelo serviço responsável pelo recurso. Validar apenas na borda é arriscado porque chamadas internas ou novos caminhos podem contornar essa verificação.

![Fluxo de autenticação e autorização](../assets/auth.png)

Depois da autenticação, a aplicação identifica o usuário. Na autorização, verifica se ele pode executar aquela ação sobre aquele recurso. Por fim, a camada de domínio executa a regra de negócio.

---

## Proteção de dados

Alguns riscos são acesso não autorizado, vazamentos, falhas de configuração, ameaças internas e integrações comprometidas. Para reduzi-los:

- **Princípio do menor privilégio:** cada usuário ou serviço recebe apenas as permissões necessárias.
- **Rotação de chaves e segredos:** credenciais devem ter ciclo de vida definido e ser armazenadas fora do código.
- **LGPD e GDPR:** coleta, retenção e uso dos dados devem respeitar as leis e os direitos aplicáveis.
- **Criptografia em trânsito:** use HTTPS/TLS nas comunicações externas e, quando necessário, entre serviços internos.
- **Criptografia em repouso:** proteja bancos, backups e objetos armazenados.
- **Auditoria:** registre ações sensíveis sem gravar senhas, tokens ou dados pessoais desnecessários.
