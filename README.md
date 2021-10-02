# ¿Cómo crear un bot de Discord con TypeScript?

Esta es una guía sencilla para guiarte y empezar a usar este increíble lenguaje, basado en JavaScript. Es uno de los mejores lenguajes que existen actualmente, mejorado por Microsoft, un superconjunto de JavaScript, que esencialmente añade tipos estáticos y objetos basados en clases.

## Cosas a tener en cuenta:

- Necesitas tener un conocimiento un poco avanzado de JavaScript, para que se te resulte fácil este lenguaje.
- El editor de código recomendado es Visual Studio Code (Vsc), **no uses Replit**, es un editor de código en línea, y no detectará los errores adecuadamente.
- No me responsabilizo si hay errores por copiar y pegar el código directamente, copia el código **a mano, línea por línea** para que no tengas ningún error.

## Empezemos

**1.** Crea una carpeta, de cualquier nombre. Es recomendable que los espacios sean "**\_**", y que no tenga mayúsculas.
**2.** Abrimos una nueva terminal con **Ctrl + Ñ**, y ejecutamos el comando **npm init -y**.
**3.** Instalamos los siguientes paquetes:

```js
npm i discord.js dotenv mongoose --save
```

**4.** Luego de eso, instalamos los paquetes internos:

```js
npm i -D typescript ts-node ts-node-dev --save
```

**5.** Creamos el archivo **tsconfig.json**, y escribimos dentro lo siguiente:

```json
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "module": "CommonJS",
    "moduleResolution": "Node",
    "target": "ESNext",
    "outDir": "dist",
    "sourceMap": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "allowSyntheticDefaultImports": true,
    "skipLibCheck": true,
    "declaration": true,
    "resolveJsonModule": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

**6.** Empezemos con el código en sí, para lo cual, creamos el archivo **.env**, dentro colocamos lo siguiente:

```env
TOKEN=El token de tu bot de Discord
MONGO=El url de Mongoose
```

Por ejemplo podría ser de esta manera: _(Los carácteres del token son generados, no es un token real)_

```env
TOKEN=ODWzHMZZtDBhXyD0qmleqUJs.D6iTCo.rBRKqbEH3yDBH7CpUTFsgerEDUT058s9we01f8we181
MONGO=mongodb+srv://user:password@testdatabase.j9pz9.mongodb.net/myFirstDatabase
```

**7.** Ahora creamos la carpeta **src**, dentro de la cual crearemos las siguientes carpetas:

- **Client**
- **Commands**
- **Events**
- **Interfaces**
- **Models**
- **Utils**

**8.** Dentro de la carpeta **src** creamos los archivos **index.ts** y **config.json**. En el **config.json** colocamos lo siguiente:

```json
{
  "prefix": "!"
}
```

Puedes colocar el prefix que gustes.

El **index.ts** lo podemos dejar vacío sin problemas, luego iremos a ello.

**8.** Dentro de la carpeta **Client**, creamos el archivo **index.ts**, para empezar a extender la clase **Client**. Escribimos el siguiente código:

```ts
import { Client, Collection } from "discord.js"; //Importamos la clase Client y Collection.
import { Command, Config } from "../Interfaces"; //Importamos las interfaces Command y Config, si les sale en rojo es normal, en el próximo paso veremos esto.
import { runAll } from "../Utils/runner"; //Importamos la función runAll, si les sale en rojo es normal, lo veremos en el paso 10.
import ConfigJson from "../config.json"; //Importamos la config.
import * as dotenv from "dotenv"; //Importamos dotenv, se hace de diferente manera en TypeScript.
dotenv.config(); //Y ejecutamos la función config del mismo.

class ExtendedClient extends Client {
  //Creamos la clase ExtendedClient y la extendemos con Client.
  constructor() {
    //Abrimos un constructor, sin parámetros.
    super({
      //Y hacemos el super, básicamente esto es lo que colocas dentro del "new Client()".
      intents: 32767, //Los intents los puedes calcular con una página, no es recomendable ponerlo de esta manera.
      partials: ["MESSAGE", "CHANNEL", "REACTION"], //Los partials son importantes para guardar cosas en caché.
      allowedMentions: {
        //Las menciones permitidas
        parse: ["users"], //Acá le decimos que sólo pueda pingear a usuarios, los roles y everyone no se mencionarán.
        repliedUser: false, //Al hacer un "message.reply" no hará ping.
      }, //Cerramos el allowedMentions.
    }); //Cerramos el super.
  } //Cerramos el constructor.
  public commands: Collection<string, Command> = new Collection(); //Definimos commands como una colección de los comandos.
  public config: Config = ConfigJson; //Definimos config como la configuración.
  public init() {
    //Definimos init como la función para iniciar el bot.
    this.login(process.env.TOKEN); //Logueamos al bot
    runAll(this); //Ejecutamos todo, eso lo veremos en el paso 10.
    return this; //Y que retorne el ExtendedCLient, si quieres hacer un comando de reload.
  } //Cerramos la función init.
} //Cerramos la clase ExtendedCLient.

export default ExtendedClient; //Exportamos la clase.
```

**9.** Vamos a la carpeta **Interfaces**, y creamos 4 archivos:

- **Config.ts**
- **Command.ts**
- **Event.ts**
- **index.ts**

Dentro de los cuales, colocaremos el siguiente contenido en cada uno: _(Reitero, no me responsabilizo de los errores si directamente haces C&P)_

- **Config.ts**

```ts
export interface Config {
  prefix: string;
}
```

- **Command.ts**

```ts
import Client from "../Client";
import { Message } from "discord.js";

interface Run {
  (client: Client, message: Message, args: string[]);
}

export interface Command {
  name: string;
  description?: string;
  aliases?: string[];
  run: Run;
}
```

- **Event.ts**

```ts
import Client from "../Client";
import { ClientEvents } from "discord.js";

interface Run {
  (client: Client, ...any: any[]);
}

export interface Event {
  name: keyof ClientEvents;
  run: Run;
}
```

- **index.ts**

```ts
export { Command } from "./Command";
export { Event } from "./Event";
export { Config } from "./Config";
```

**10.** Dentro de la carpeta **Utils**, creamos el archivo **runner.ts**, y colocamos lo siguiente:

```ts
import Client from "../Client"; //Importamos el ExtendedCLient.
import { readdirSync } from "fs";
import { Command, Event } from "../Interfaces";
import { connect } from "mongoose";

export function runAll(client: Client) {
  //Exportamos la función runAll.

  //Commands
  readdirSync("./src/Commands/Cmd").forEach((dir) => {
    //Entramos a la carpeta Commands, creamos la carpeta Cmd, y hacemos un readdirSync, para obtener todas sus categorias, retornando el parámetro dir
    readdirSync("./src/Commands/Cmd/" + dir) //Luego de eso entramos al mismo directorio, pero con el parámetro dir añadido.
      .filter((f) => f.endsWith(".ts")) //Filtro de sólo archivos TypeScript.
      .forEach((file) => {
        //Y un forEach.
        let { command } = require(`../Commands/Cmd/${folder}}/${file}`); //Requerimos el comando de los comandos.
        client.commands.set((command as Command).name, command as Command); //Y lo establecemos en el Collection.
      }); //Cerramos forEach de archivos.
  }); //Cerramos forEach de categorías

  //Events
  readdirSync("./src/Events/Djs") //Entramos a la carpeta Events, creamos la carpeta Djs, y hacemos un readdirSync para obtener todos sus archivos.
    .filter((f) => f.endsWith(".ts")) //Filtro de sólo archivos TypeScript.
    .forEach((file) => {
      //Y un forEach.
      let { event } = require("../Events/Djs/" + file); //Requerimos el evento de los eventos.
      client.on((event as Event).name, (event as Event).run.bind(null, client)); //Y lo guardamos en caché, para que se emitan al cumplirse cada evento.
    }); //Cerramos forEach.

  //unhandledRejection = Básicamente esto sirve para que tu bot no se apague al instante si hay un error, sólo enviará un error a la consola, más no lo apagará.
  process.on("unhandledRejection", async (rejection: PromiseRejectedResult) => {
    //Ejecutamos el evento con el parámetro rejection.
    logReject(); //Ejecutamos la función logReject.
    function logReject() {
      //Abrimos la función
      console.log("Unhandled Rejection:", rejection); //Y enviamos a la consola el error.
      process.stdout.clearLine(1); //Eliminamos una línea de la consola.
    } //Cerramos la función.
  }); //Cerramos el evento.

  //Mongoose
  connect(process.env.MONGO, {
    useCreateIndex: true,
    useFindAndModify: false,
    useNewUrlParser: true,
    useUnifiedTopology: true,
  }) //Conectamos con Mongose, si les sale en rojo, intenten solo dejando "connect(process.env.MONGO)".
    .then(() => console.log("✅ | Conectado a Mongoose")) //Si todo salió bien, envía el mensaje a la consola.
    .catch(
      //De lo contrario
      (
        e //Parámetro "e", de error.
      ) => console.error("❌ | Ocurrió un error al conectarme a Mongoose:", e) //Si hay un error, envía el mensaje y error al mismo.
    ); //Cerramos catch
} //Cerramos la función exportada, runALl.
```

**11.** En la carpeta **Events/Djs**, creamos 2 archivos:

- **ready.ts**
- **messageCreate.ts**

En cada uno colocamos lo siguiente:

**ready.ts**

```ts
import { Event } from "../../Interfaces"; //Importamos la interfaz Event.

export const event: Event = {
  //Exportamos el evento.
  name: "ready",
  run: async (client) => {
    //Y lo ejecutamos.
    console.log("¡Estoy listo!"); //Enviamos a la consola el mensaje de que inició correctamente.
  }, //Cerramos run.
}; //Y cerramos el evento exportado.
```

**messageCreate.ts**

```ts
import { Message } from "discord.js"; //Importamos la clase Message.
import { Command, Event } from "../../Interfaces"; //Importamos las interfaces Command y Event.

export const event: Event = {
  //Exportamos el evento.
  name: "messageCreate",
  run: async (client, message: Message) => {
    //Y lo ejecutamos
    if (
      //Si...
      message.author.bot || //El autor es un bot
      message.channel.type == "DM" || //El tipo de canal es un dm
      !message.guild || //Si no hay servidor
      !message.content.toLowerCase().startsWith(client.config.prefix) //O si no inicia con el prefix
    )
      return; //Retorna.
    let args = message.content
      .slice(client.config.prefix.length)
      .trim()
      .split(/ +/g); //Definimos los args.
    let command = args.shift().toLowerCase(); //Definimos el comando.
    let cmd = client.commands.find(
      //Buscamos el comando.
      (x) => x.name == command || (x.aliases && x.aliases.includes(command)) //Si encuentra el comando por el nombre o por el alias.
    );
    try {
      //Si no hay errores.
      cmd.run(client, message, args); //Ejecutamos el comando.
    } catch (e) {
      //De lo contrario.
      message.reply("Ocurrió un error, revisa la consola."); //Mensaje.
      return console.error(e); //Error a la consola.
    } //Cerramos catch.
  }, //Cerramos run.
}; //Ceramos el evento exportado.
```

**12.** Vamos a hacer un comando, la estructura es la siguiente:

```ts
import { Command } from "../../../Interfaces"; //Importamos la interfaz Command.

export const command: Command = {
  //Exportamos el comando
  name: "", //Nombre del comando.
  description: "", //Opcional, la descripción del mismo.
  aliases: [], //Opcional, los alias del mismo.
  run: async (client, message, args) => {
    //Ejecutamos el comando
  }, //Cerramos run.
}; //Cerramos el comando exportado
```

Hagamos un comando de ejemplo, en este caso **ping**:

```ts
import { Command } from "../../../Interfaces";

export const command: Command = {
  name: "ping",
  aliases: [],
  run: async (client, message, args) => {
    message.reply("¡Pong!");
  },
};
```

**13.** Vamos al archivo **package.json**, y creamos un script con lo siguiente:

```json
{
  "start": "ts-node-dev --respawn --transpile-only --poll ./src/index.ts"
}
```

<img src='https://i.imgur.com/zZzecF8.png'></img>

**14.** ¿Recuerdas el archivo **index.ts** vacío, a la altura del **config.json**? Pues dentro colocaremos lo siguiente:

```ts
import Client from "./Client";
new Client().init();
```

**15.** Por último, abre nuevamente la terminal con **Ctrl + Ñ**, y ejecuta **npm start**. El bot debería iniciar correctamente.

Si hiciste todos los pasos correctamente, el bot debería de iniciar sin problemas.
Hasta aquí llega esta guía para usar TypeScript para un bot de Discord, espero te haya servido :D

<img src="https://i.imgur.com/ipDelAP.png"></img>
