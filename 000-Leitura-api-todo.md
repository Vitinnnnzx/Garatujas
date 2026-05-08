
*CORE.TS*
```typescript
const jsonFilePath = __dirname + '/data.temp.json'; 
const list: string[] = await loadFromFile();


async function loadFromFile() {
  try {
    const file = Bun.file(jsonFilePath);
    const content = await file.text();
    return JSON.parse(content) as string[];
  } catch (error: any) {
    if (error.code === 'ENOENT')
      return [];
    throw error;
  }
}


async function saveToFile() {
  try {
    await Bun.write(jsonFilePath, JSON.stringify(list));
  } catch (error: any) {
   throw new Error("Erro ao salvar os dados no arquivo: " + error.message);
  }
}


async function addItem(item: string) {
  list.push(item);
  await saveToFile();
}


async function getItems() {
  return list;
}


async function updateItem(index: number, newItem: string) {
  if (index < 0 || index >= list.length)
    throw new Error("Index fora dos limites");
  list[index] = newItem;
  await saveToFile();
}


async function removeItem(index: number) {
  if (index < 0 || index >= list.length)
    throw new Error("Index fora dos limites");
  list.splice(index, 1);
  await saveToFile();
}


export default { addItem, getItems, updateItem, removeItem };
```

---

*API*
```typescript
import todo from "./core.ts"; // pega as function do core

const server = Bun.serve({ // iniciliza o servidor
  port: 3000, // determina a porta, pensando q o server é um predio, a port é o andar

  routes: { //abre a rotas que cada metodo utilizara
    "/": new Response(Bun.file("./public/index.html")), // rota pra puxar o html 

    "/api/todo": { //rota
      GET: async () => { // arrow function assincrona para "listar"
        const items = await todo.getItems() // variavel q espera realizar o getitems( que veio do core.ts 
        return Response.json(items) // retorna o valor que pego no json
      },

      POST: async (req) => { // arrow function assincrona que pega a req
        const data = await req.json() as any; // variavel que espera e tranforma o valor dado em json
        const item = data.item || null; // tranforma o valor em string 
        if (!item) // verifca se esse dado é um item
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 }); // se n for retorna esse erro
        await todo.addItem(item); // se for ele espera e adicona 
        return Response.json(data); //retorna o valor para o json
      },
    },

    "/api/todo/:index": { // rota com index q qr listar  ou deletar
      PUT: async (req) => { arrow function assincrona que updt a req
        const index = parseInt(req.params.index); // ???
        if (isNaN(index)) // verifca se o valor é um numero
          return Response.json('Índice inválido. um número inteiro é esperado.', { status: 400 }); // se n for retorna erro
        const data = await req.json() as any; / variavel que espera e tranforma o valor dado em json
        const newItem = data.newItem || null; tranforma o novo valor em string 
        if (!newItem) //verifica se tem um novo valor 
          return Response.json('Por favor, forneça um novo item para atualizar.', { status: 400 }); // se n for retorna esse erro
        try { // tenta KKKK traduçao literal
          await todo.updateItem(index, newItem); // faz o update, pegando o index e pelo oque deve atualizar
          return Response.json(`Item no índice ${index} atualizado para "${newItem}".`); // retornma msg bonita falando q deu certo
        } catch (error: any) { // se a tentavia de algum erro vem pra ca
          return Response.json(error.message, { status: 400 }); // retorna um erro, pq o usuariio é tanso
        }
      },

      DELETE: async (req) => { arrow function assincrona que delete a req
        const index = parseInt(req.params.index); //???
        if (isNaN(index)) // verifca se o valor é um numero
          return Response.json('Índice inválido.', { status: 400 }); //se n for retorna erro 
        try { // tenta
          await todo.removeItem(index); // pega o iindex para remover, puxa o remove la do core
          return Response.json(`Item no índice ${index} removido com sucesso.`); // retornma msg bonita falando q deu certo
        } catch (error: any) { // se a tentavia de algum erro vem pra ca
          return Response.json(error.message, { status: 400 }); // retorna um erro, pq o usuariio é tanso
        }
      },
    },
  },

  async fetch(req) { // se nd der certo vem aq
    return new Response(`Not Found`, { status: 404 });  // retorna um erro pq n acho
  },
});

console.log(`Server running at http://localhost:${server.port}`); // fala pra gnt se ta rodando na port certa
```
