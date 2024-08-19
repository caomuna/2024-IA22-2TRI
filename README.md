# Iniciando um projeto Node.js com TypeScript 

1. Faça um novo repositório clicando no “+” localizado à direita da tela. 
2. Crie um diretório e o utilize pelo codespace do GitHub. 

## Node.js no codespace

### Abra o terminal, cole e execute um por um dos seguintes códigos:
```
npm init -y 
npm install express cors sqlite3 sqlite 
npm install --save-dev typescript nodemon ts-node @types/express @types/cors 
npx tsc --init 
mkdir src 
touch src/app.ts
```

### Crie um arquivo nomeado ``.gitignore`` e adicione o seguinte texto 
```
node_modules/ 
dist/

Database.sqlite
```

## tsconfig.json 
Abra o arquivo do tsconfig.json, com “ctrl+F” subistitua a linha ``//outDir": "./`` por ``outDir": "./dist``. Faça o mesmo com a linha ``"rootDirs": []`` a subistitua por ``rootDir": "./src``

## package.json 
Agora no package.json, subistitue a linha ``"test": "echo \"Error: no test specified\" && exit 1" }`` que está abaixo de “scripts” por``"dev": "npx nodemon src/app.ts"`` 

## Primeiro arquivo 
Na pasta ``scr``, crie o arquivo chamado ``scr/app.ts`` e adicione o código abaixo  
````
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
````

## run do server 
Agora no terminal cole o seguinte comando  
````
npm run dev
````
Caso tenha seguido os passos corretos, assim espero, a mensagem de ``Server running on port 3333`` sera escrita no terminal. 

### teste 
Para realmente saber se tudo ocorreu bem clique no linke que se localiza em portas ao lado do terminal, o seguinte texto tem que ser visivel “Hello Word”.

## Banco de dados
Na pasta do ``scr`` crie um arquivo de nome ``database.ts`` e cole o seguinte codigo
````
import { open, Database } from 'sqlite'; 
import sqlite3 from 'sqlite3'; 
let instance: Database | null = null; 
export async function connect() { 
  if (instance) return instance; 
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
````
## Adicionar o banco ao server 
Para isso será necessário substituir o código do arquivo ``scr/app.ts`` pelo seguinte código. 
````
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
`````
## teste de dados 
Antes de tudo no termina do codespece a porta tem que está aberta para isso basta apenas um clique com o botão direito. 

Para inserimos algo no banco temos que fazer um post, no caso recomendo a utilização do https://www.postman.com/ que pode ser utilizado online, crie uma conta crie também um “New Collection” e um “New Request”.  

Nesta nova tela clique em Get e troque por Post, junto a isto abra o Body selecione a opção Raw. 
Após isto cole o seguinte código  
````
{ 
  "name": "John Doe", 
  "email": "johndoe@mail.com" 
} 
````
Clique em Send e se estiver tudo correto você terá a resposta  
````
{ 
  "id": 1, 
  "name": "John Doe", 
  "email": "johndoe@mail.com" 
} 
````
## Lista de usuarios
Adicione a rota ``/users`` ao servidor, e no arquivo ``scr/app.ts`` cole o seguinte código. 
````
app.get('/users', async (req, res) => { 
  const db = await connect(); 
  const users = await db.all('SELECT * FROM users'); 
  res.json(users); 
});
````
## Edição/Exclução de usuário 
Para estes comandos basta apenas adicionar a rota de ``/users/:id`` ao servidor. 
### Edição
````
app.put('/users/:id', async (req, res) => { 
  const db = await connect(); 
  const { name, email } = req.body; 
  const { id } = req.params; 
  await db.run('UPDATE users SET name = ?, email = ? WHERE id = ?', [name, email, id]); 
  const user = await db.get('SELECT * FROM users WHERE id = ?', [id]); 
  res.json(user); 
});
````
### Exclução
````
app.delete('/users/:id', async (req, res) => { 
  const db = await connect(); 
  const { id } = req.params; 
  await db.run('DELETE FROM users WHERE id = ?', [id]); 
  res.json({ message: 'User deleted' }); 
});
````









