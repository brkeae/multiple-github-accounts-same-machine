# Trabalhando com múltiplas contas do Git em uma mesma máquina

### Pré-requisito
- É necessário que você tenha o Git instalado em sua máquina.
- Nesse tutorial, será utilizado um ambiente Linux (Ubuntu 22.04).

### Passo a passo

### 1. Crie as chaves SSH e as configure nas contas do Git

Para trabalhar com acesso SSH no Git, vocẽ precisa criar as chaves localmente através do comando 
"ssh-keygen", no exemplo abaixo, vamos entrar no diretório oculto ".ssh" e criar duas chaves, partindo
do princípio que teremos duas contas de natureza distintas na máquina, uma para uso pessoal (personal) e 
outra de uso profissional (work).

```
    $ cd ~/.ssh
    $ ssh-keygen -t ed25519 -C "email-pessoal@provedor.com"
    $ ssh-keygen -t ed25519 -C "email-profissional@provedor.com"
```
Ao executar esse comando, o terminal irá perguntar qual o nome do arquivo você deseja salvar com as chaves
e também irá propor que vocẽ inclua uma senha para essas chaves, mas a senha é opcional.

Salve os arquivos com nomes diferentes, por exemplo:

```
- personal_id_ed25519
- work_id_ed25519
```

Após essa etapa, você perceberá que terão 4 arquivos no diretório ".ssh/":

- personal_id_ed25519
- personal_id_ed25519.pub
- work_id_ed25519
- work_id_ed25519.pub

Vamos pegar o conteúdo dos arquivos ".pub" para configurar as contas no site do Git.

```
$ cat work_id_ed25519.pub
```
O terminal irá exibir algo como:
```
ssh-ed25519 AAAAC3C1lEfdsf5AAFDAVVDAAAFDIIgbNZ+TCGfurreeoUhwy... email-pessoal@provedor.com
```

Copie esse valor e vá até o Github da conta respectiva
e adicione essa chave nas configurações da conta "SSH and GPG Keys" > "New SSH Key"




### 2. Crie os arquivos Git de configuração

Com esses arquivos iremos dizer ao Git o nome do usuário e o e-mail que está operando nos diretórios
de cada conta que está sendo controlada pelo Git local.

  ```
    $ cd ~
    $ touch .gitconfig .gitconfig-personal .gitconfig-work
  ```
Edite o arquivo .gitconfig e coloque o seguinte conteúdo (adaptando para seus diretórios locais).
Esse arquivo será responsável para dizer ao Git local qual arquivo de configuração ele deve utilizar para cada diretório que está associado a cada uma
das contas que vocẽ está configurando.

```
[includeIf "gitdir:~/git/personal/"]
    path = ~/.gitconfig-personal
[includeIf "gitdir:~/git/work/"]
    path = ~/.gitconfig-work
```

Edite os arquivos .gitconfig-personal e .gitconfig-work com as informações de usuário e e-mail de cada conta.

**.gitconfig-personal**
```
[user]
    name = usuario-pessoal
    email = email-pessoal@provedor.com
```
**.gitconfig-work**
```
[user]
    name = usuario-work
    email = email-profissional@provedor.com
```

### 3. Crie os diretórios

Dentro do diretório "work" você irá realizar o clone dos repositórios referentes ao trabalho, e no diretório "personal" os repositórios pessoais.

```
    $ cd ~
    $ mkdir git
    $ cd git
    $ mkdir personal work
```


### 4. Configurando o SSH

Por fim, iremos dizer ao SSH, qual chave ele deverá utilizar para se conectar com o Git remoto (origin) de cada conta.

```
    $ cd ~/.ssh
    $ nano config
```

Cole o conteúdo abaixo e salve o arquivo.

```
# Conta GitHub Pessoal
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/personal_id_ed25519

# Conta GitHub Trabalho
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/work_id_ed25519
```

Tendo feito isso, observe o seguinte, sempre que for clonar um repositório de "work" o seu host deverá ser "github-work", já para
os repositórios pessoais, deixamos github.com por padrão, mas você pode customizar isso.

Exemplo de um repositório da conta "work" sendo clonado:

```
$ cd ~/git/work

$ git clone git@github-work.com:teste/teste.git
```

Exemplo de um repositório da conta "personal" sendo clonado:

```
$ cd ~/git/personal

$ git clone git@github.com:teste/teste.git
```

### 5. Testando conectividade SSH

Para validar as configurações das chaves dentro de cada diretório, você pode realizar um teste SSH no Git remoto com o comando:

```
ssh -T git@github.com
```
Aqui você deverá receber uma mensagem de conexão com sucesso ao Git da conta pessoal
```
ssh -T git@github-work.com
```
Aqui você deverá receber uma mensagem de conexão com sucesso ao Git da conta work

### 5. Testando as configurações de user e e-mail do Git

Para validar que os arquivos de configuração do Git estão corretos, entre em algum repositório clonado e digite o comando:

```
git config user.name
git config user.email
```
O retorno deverá ser o e-mail e usuário que vocẽ configurou no segundo passo nos arquivos .gitconfig-personal e .gitconfig-work