# Iniciando um projeto Node.js com TypeScript

### Faça uma conta no GitHub, caso ainda não tenha uma. Depois, siga os passos abaixo:

1. Clique no ícone "+" no canto superior direito e selecione **New repository**.
2. Escolha um nome para o repositório e marque a opção **Add a README file**. Finalize clicando no botão verde.

### Criando o projeto

1. Crie um diretório para o projeto e acesse-o pelo Codespace do GitHub:
   - Clique em **Code**, depois em **Codespaces**, e crie um novo Codespace clicando no botão **+**.
   - Para reabrir o Codespace posteriormente, siga o mesmo procedimento, mas dessa vez clique no nome do Codespace abaixo de **On current branch** (o nome terá um pequeno `main*` abaixo).

2. Abra o terminal no Codespace (pressione `Ctrl + Aspas (")`) e execute os seguintes comandos para instalar o Node.js:

    npm init -y
    npm install express cors sqlite3 sqlite
    npm install --save-dev typescript nodemon ts-node @types/express @types/cors
    npx tsc --init
    mkdir src
    touch src/app.ts

3. Crie um arquivo `.gitignore` na raiz do projeto com o seguinte conteúdo:

    node_modules/
    dist/
    database.sqlite

### Configurando o `tsconfig.json`

Abra o arquivo `tsconfig.json` e faça as seguintes alterações:

- Substitua `"outDir": "./",` por `"outDir": "./dist",`.
- Substitua `"rootDirs": [],` por `"rootDir": "./src",`.

O arquivo ficará assim:

    {
      "compilerOptions": {
        "target": "ES2017",
        "module": "commonjs",
        "rootDir": "./src",
        "outDir": "./dist",
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "strict": true
      }
    }

### Configurando o `package.json`

Abra o arquivo `package.json` e substitua o script `"test": "echo \"Error: no test specified\" && exit 1"` pelo seguinte:

    "scripts": {
      "dev": "npx nodemon src/app.ts"
    },

### Criando o arquivo inicial do servidor

Dentro da pasta `src`, crie um arquivo chamado `app.ts` com o seguinte conteúdo:

    import express from 'express';
    import cors from 'cors';

    const port = 3333;
    const app = express();

    app.use(cors());
    app.use(express.json());

    app.get('/', (req, res) => {
      res.send('Hello World');
    });

    app.listen(port, () => {
      console.log(`Server running on port ${port}`);
    });

### Inicializando o servidor

No terminal, execute:

    npm run dev

Se tudo estiver correto, você verá a mensagem `Server running on port 3333` no terminal.

### Testando o servidor

Abra o navegador e acesse `http://localhost:3333` ou, após executar o comando `npm run dev`, clique na opção "abrir no navegador" que aparecerá no canto inferior direito.

### Configurando o banco de dados

Crie um arquivo chamado `database.ts` dentro da pasta `src` e adicione o seguinte código:

    import { open, Database } from 'sqlite';
    import sqlite3 from 'sqlite3';

    let instance: Database | null = null;

    export async function connect() {
      if (instance !== null) 
          return instance;

      const db = await open({
         filename: './src/database.sqlite',
         driver: sqlite3.Database
      });

      await db.exec(`
        CREATE TABLE IF NOT EXISTS users (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          name TEXT,
          email TEXT
        )
      `);

      instance = db;
      return db;
    }

### Adicionando o banco de dados ao servidor

Substitua o código do arquivo `src/app.ts` pelo seguinte:

    import express from 'express';
    import cors from 'cors';
    import { connect } from './database';

    const port = 3333;
    const app = express();

    app.use(cors());
    app.use(express.json());

    app.get('/', (req, res) => {
      res.send('Hello World');
    });

    app.get('/users', async (req, res) => {
      const db = await connect();
      const users = await db.all('SELECT * FROM users');
      res.json(users);
    });

    app.post('/users', async (req, res) => {
      const db = await connect();
      const { name, email } = req.body;
      const result = await db.run('INSERT INTO users (name, email) VALUES (?, ?)', [name, email]);
      const user = await db.get('SELECT * FROM users WHERE id = ?', [result.lastID]);
      res.json(user);
    });

    app.listen(port, () => {
      console.log(`Server running on port ${port}`);
    });

### Testando com o Postman

1. Baixe e instale o Postman em [postman.com](https://www.postman.com).
2. Crie uma conta, abra o Postman, clique em **+** e crie uma **New Collection** e depois um **New Request**.
3. Altere o método para **POST** e, em **Body**, selecione **raw** e escolha **JSON**.

### Testando a inserção de dados

Faça uma requisição POST para `http://localhost:3333/users` com o seguinte corpo JSON:

    {
      "name": "John Doe",
      "email": "johndoe@mail.com"
    }

Lembre-se de tornar a porta do localhost pública no Codespace. Após isso, clique em **Send** no Postman. Se tudo der certo, você verá a resposta com o usuário inserido:

    {
      "id": 1,
      "name": "John Doe",
      "email": "johndoe@mail.com"
    }

### Listando os usuários

Adicione a seguinte rota no servidor, abaixo do `res.json(user);`:

    app.get('/users', async (req, res) => {
      const db = await connect();
      const users = await db.all('SELECT * FROM users');
      res.json(users);
    });

### Editando um usuário

Adicione a rota `/users/:id` ao servidor:

    app.put('/users/:id', async (req, res) => {
      const db = await connect();
      const { name, email } = req.body;
      const { id } = req.params;
      await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]);
      const user = await db.get('SELECT * FROM users WHERE id = ?', [id]);
      res.json(user);
    });

### Deletando um usuário

Adicione a rota `/users/:id` ao servidor:

    app.delete('/users/:id', async (req, res) => {
      const db = await connect();
      const { id } = req.params;

      await db.run('DELETE FROM users WHERE id = ?', [id]);

      res.json({ message: 'User deleted' });
    });

### Criando o `index.html`

Crie uma pasta chamada `public` e dentro dela, crie um arquivo chamado `index.html`. Adicione o seguinte código:

    <!DOCTYPE html>
    <html lang="en">

    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>

    <body>
      <form>
        <input type="text" name="name" placeholder="Nome">
        <input type="email" name="email" placeholder="Email">
        <button type="submit">Cadastrar</button>
      </form>

      <table>
        <thead>
          <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Email</th>
            <th>Ações</th>
          </tr>
        </thead>
        <tbody>
          <!--  -->
        </tbody>
      </table>

      <script>
        const form = document.querySelector('form')

        form.addEventListener('submit', async (event) => {
          event.preventDefault()

          const name = form.name.value
          const email = form.email.value

          await fetch('/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ name, email })
          })

          form.reset()
          fetchData()
        })

        const tbody = document.querySelector('tbody')

        async function fetchData() {
          const resp = await fetch('/users')
          const data = await resp.json()

          tbody.innerHTML = ''

          data.forEach(user => {
            const tr = document.createElement('tr')
            tr.innerHTML = `
              <td>${user.id}</td>
              <td>${user.name}</td>
              <td>${user.email}</td>
              <td>
                <button class="excluir">Excluir</button>
                <button class="editar">Editar</button>
              </td>
            `

            const btExcluir = tr.querySelector('button.excluir')
            const btEditar = tr.querySelector('button.editar')

            btExcluir.addEvent

Agora você pode copiar e colar este conteúdo diretamente no seu arquivo markdown no GitHub!
