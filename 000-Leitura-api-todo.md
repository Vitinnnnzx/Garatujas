
### *CORE.TS*
```typescript
const jsonFilePath = __dirname + '/data.temp.json'; //pega o diretorio do arquivo, a loc exata dele e a a parte do json
const list: string[] = await loadFromFile(); // cria um a lista, basicamente um array de string, que depois ??
// função assincorna para carregar o que tem na pasta
async function loadFromFile() {
  try { // tenta fazer, qualquer erro interrompe
    const file = Bun.file(jsonFilePath); // variavel que pega oque carrega do json
    const content = await file.text(); // variavel pra pegar o texto que tem na pasta
    return JSON.parse(content) as string[];  // transforma dados do jsno em string array
  } catch (error: any) { // se a tentativa der erro, vem aqui
    if (error.code === 'ENOENT')// se der erro
      return []; // retorna um array vazio a pasta
    throw error; // joga o erro
  }
}
// função assincorna para salvar
async function saveToFile() { 
  try { // tenta fazer, qualquer erro interrompe
    await Bun.write(jsonFilePath, JSON.stringify(list)); // ???
  } catch (error: any) { // se der erro vem aq
   throw new Error("Erro ao salvar os dados no arquivo: " + error.message); // joga uma msg de erro
  }
}
//funcao q adiciona itens 
async function addItem(item: string) {
  list.push(item); // adiciona no array o item dado
  await saveToFile(); // utiliza a funçao pra salvar
}
//funcao para listar os items
async function getItems() {
  return list; // so puxa o array
}
// funçao para dar update , puxa o index e o valor
async function updateItem(index: number, newItem: string) { 
  if (index < 0 || index >= list.length) // se o index for menor q 0 ou maior q o numero do array
    throw new Error("Index fora dos limites"); // joga uma msg de erro de index
  list[index] = newItem; // // substitui dentro do array o valor no index indicado
  await saveToFile(); // salva ne
}
// funçao que remove/ deleta, puxa o index so
async function removeItem(index: number) {
  if (index < 0 || index >= list.length) // se o index for menor q 0 ou maior q o numero do array
    throw new Error("Index fora dos limites");  // joga uma msg de erro de index
  list.splice(index, 1); // deleta o item do index indicado
  await saveToFile(); // salva ne
}

export default { addItem, getItems, updateItem, removeItem }; // exporta para fora as funções 
```

---

### *API*
```typescript
import todo from "./core.ts"; // pega as function do core

// iniciliza o servidor
const server = Bun.serve({ 
  port: 3000, // determina a porta, pensando q o server é um predio, a port é o andar

  //abre a rotas que cada metodo utilizara
  routes: { 
    "/": new Response(Bun.file("./public/index.html")), // rota pra puxar o html !fixo!
      //path
    "/api/todo": { 
      GET: async () => { // arrow function assincrona para "listar"
        const items = await todo.getItems() // variavel q espera realizar o getitems( que veio do core.ts 
        return Response.json(items) // retorna o valor que pego no json
      },
      // arrow function assincrona que pega a req
      POST: async (req) => { 
        const data = await req.json() as any; // variavel que espera e tranforma o valor dado em json
        const item = data.item || null; // tranforma o valor em string 
        if (!item) // verifca se esse dado é um item
          return Response.json('Por favor, forneça um item para adicionar.', { status: 400 }); // se n for retorna esse erro
        await todo.addItem(item); // se for ele espera e adicona 
        return Response.json(data); //retorna o valor para o json
      },
    },
    // path apos rota com index q qr listar  ou deletar
    "/api/todo/:index": { 
      PUT: async (req) => { arrow function assincrona que updt a req
        const index = parseInt(req.params.index); // transforma oque estiver no index, que esta como string, e "tenta" tranforma em numero inteiro
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
      //arrow function assincrona que delete a req
      DELETE: async (req) => { 
        const index = parseInt(req.params.index); // transforma oque estiver no index, que esta como string, e "tenta" tranforma em numero inteiro
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
  // se nd der certo vem aq
  async fetch(req) { 
    return new Response(`Not Found`, { status: 404 });  // retorna um erro pq n acho
  },
});

console.log(`Server running at http://localhost:${server.port}`); // fala pra gnt se ta rodando na port certa
```
