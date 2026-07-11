# Blob Store (Object Storage)

Quando projetamos a arquitetura de um sistema (System Design), **nunca** devemos armazenar arquivos grandes (mídias, PDFs, binários) diretamente em um banco de dados relacional ou convencional. Isso ocorre devido a vários problemas de escala:

- **Inflação do Banco de Dados:** O tamanho do banco cresce exponencialmente, tornando backups e replicações extremamente lentos e complexos.
- **Degradação de Performance:** Queries ficam lentas porque as páginas de dados ficam sobrecarregadas, exigindo mais I/O do disco para ler registros.
- **Desperdício de Recursos na Escala:** Para escalar leituras usando réplicas de banco de dados, você acabaria duplicando arquivos pesados desnecessariamente em servidores caros.
- **Custo Ineficiente:** Armazenar dados brutos em discos de alta performance de bancos de dados (como SSDs provisionados) é infinitamente mais caro do que usar storages dedicados.

Para resolver isso, separamos o armazenamento de arquivos da lógica de dados estruturados usando duas abordagens principais: **File System** e **Blob Store**.

---

## 1. File System (Sistemas de Arquivos)

Na AWS, o principal exemplo é o **EFS (Elastic File System)** ou o **EBS (Elastic Block Store)**. 

- **Como funciona:** Organiza os dados em uma estrutura **hierárquica tradicional** (árvore de pastas e subpastas) e é montado diretamente no Sistema Operacional do servidor.
- **Limitações em System Design:** É mais difícil e caro de escalar horizontalmente para bilhões de arquivos. Conforme o volume cresce, a navegação na árvore de diretórios gera gargalos de performance, além de exigir o gerenciamento de permissões complexas (POSIX) entre múltiplos servidores.

---

## 2. Blob Store (Object Storage)

Na AWS, o padrão absoluto de mercado é o **S3 (Simple Storage Service)**. 

Ao contrário do File System, ele utiliza uma **arquitetura plana (flat)**: não existem pastas reais, apenas chaves textuais (IDs únicos) que apontam diretamente para o arquivo (o Objeto).

Cada objeto em um Blob Store é composto por três partes essenciais:
- **Object ID (Key):** Um identificador único (geralmente um UUID ou um caminho/string como `uploads/user_123/foto.png`).
- **Dados (O BLOB):** O arquivo binário bruto (Fotos, Vídeos, Documentos, Áudios, Logs, arquivos `.json`, `.csv`, etc.).
- **Metadados:** Informações sobre o arquivo (como `content-type`, `created-at`, chaves de criptografia ou tags customizadas).

### Vantagens Principais para System Design:
- **Escalabilidade Horizontal Infinita:** Como o acesso é feito diretamente via chave (chave-valor), o sistema busca o arquivo sem precisar percorrer uma árvore de diretórios. A performance permanece constante se você tiver 10 ou 10 bilhões de arquivos.
- **Custo Altamente Otimizado:** O custo por GB armazenado é extremamente baixo se comparado a qualquer outra solução.
- **Acesso Descentralizado via API:** Os arquivos são expostos via HTTP/HTTPS (APIs REST). Isso permite que o cliente final (App/Web) faça o download do arquivo ou upload (via *Presigned URLs*) diretamente do S3, sem sobrecarregar os servidores da sua aplicação.

---

## 3. Estratégias de Persistência e Upload em System Design

Ao desenhar a arquitetura com Blob Stores, existem duas decisões fundamentais que precisam ser tomadas: **como referenciar o arquivo no Banco de Dados** e **como o arquivo chega até o bucket**.

### A. Referência no Banco de Dados: URL Completa vs. Apenas o Path/Key

O banco de dados relacional (ou NoSQL) continuará guardando a referência do arquivo. Existem duas abordagens comuns para isso:

1. **Salvar a URL Absoluta (Link Completo):** - *Como funciona:* Salva-se a string inteira no banco (ex: `https://meubucket.s3.amazonaws.com/uploads/UUID.png`).
* *Prós:* O backend ou o frontend só precisa ler o campo e já pode renderizar ou usar o link direto.
* *Contras:* Se você precisar migrar de provedor (ex: ir para o Azure Blob Storage) ou mudar o nome do bucket, terá que rodar um script para atualizar milhões de linhas no banco de dados.


2. **Salvar apenas o Path/Key (Recomendado):**
* *Como funciona:* Salva-se apenas o identificador/caminho relativo (ex: `uploads/UUID.png`). A URL base do bucket (`https://meubucket.s3.amazonaws.com/`) fica armazenada como uma **variável de ambiente** da aplicação.
* *Prós:* Alta flexibilidade. Se o bucket mudar, você altera apenas uma linha de configuração no ambiente e o backend se encarrega de concatenar a URL base com a chave antes de enviar para o cliente.

---

### B. Estratégia de Upload: Presigned URLs (URLs Pré-Assinadas)

Em arquiteturas tradicionais, o cliente envia o arquivo para o seu backend, e o seu backend faz o upload para o S3. Isso gera gargalos massivos de memória, rede e processamento no seu servidor (I/O Bound).

A estratégia moderna para larga escala é o uso de **Presigned URLs**.

* **O Fluxo Eficiente (Passo a Passo):**
1. O cliente (App/Web) avisa ao seu Backend: *"Quero fazer upload de uma foto chamada perfil.jpg"*.
2. O seu Backend (que possui as credenciais seguras da AWS) faz uma requisição rápida ao S3 solicitando uma **URL Pré-Assinada** de `PUT` para aquela chave específica, definindo um tempo curto de expiração (ex: 5 minutos).
3. O S3 retorna essa URL temporária e autenticada para o seu Backend, que a repassa para o Cliente.
4. O Cliente faz o upload do arquivo binário **diretamente para o S3** através dessa URL, sem trafegar nenhum byte do arquivo pelo seu servidor.
5. Após o sucesso do upload, o cliente avisa o backend para salvar a chave do objeto no banco de dados.


* **Vantagens em System Design:** - **Economia de Recursos:** Reduz drasticamente a carga de rede e CPU dos seus servidores de aplicação.
* **Segurança:** O cliente nunca tem acesso às credenciais globais do seu bucket, apenas a uma permissão temporária e restrita para um único arquivo.

---

## Conclusão / Casos de Uso

Em System Design, toda vez que o requisito do sistema envolver armazenamento massivo de arquivos não estruturados, a escolha padrão será um **Blob Store**. 

Exemplos práticos:
- **Instagram:** Armazenamento de fotos e stories dos usuários.
- **Netflix / YouTube:** Armazenamento dos arquivos de vídeo originais antes do processamento (transcoding).
- **Spotify:** Armazenamento dos arquivos de áudio das músicas e podcasts.
- **Plataformas SaaS:** Armazenamento de PDFs de notas fiscais, relatórios gerados ou logs históricos do sistema.
