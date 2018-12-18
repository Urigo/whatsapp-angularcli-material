# Step 6: graphql-code-generator

[//]: # (head-end)


## Code Generation

GraphQL entities are defined as static and typed, which means they can be analyzed and use as a base for generating everything.

There are many tools related to that topic, but we're going to focus on The **GraphQL Coge Generator**.

The GraphQL Coge Generator can generate any code for any language  —  including type definitions, data models, query builder, resolvers, ORM code, complete full stack platforms.

The tool is based on the concept of plugins. It has few official plugins to solve most required features. The GraphQL Code Gen allows to create your own custom codegen plugin in 10 minutes, that fit exactly your needs.

### Generating types for server-side code

First, let's install `graphql-code-generator`:

    yarn add graphql-code-generator

GraphQL Code Generator lets you setup everything by simply running the following command:

    yarn gql-gen init

Question by question, it will guide you through the whole process of setting up a schema, selecting and intalling plugins, picking a destination of a generated file and a lot more.

First, it will ask you what is the type of application you're going to build, pick "Backend".
When it asks you for the schema, point it to `./schema/typeDefs.ts`.
The output path should be: `./types.d.ts`

Plugins that you want to have selected are:

- TypeScript Common
- TypeScript Server
- TypeScript Resolvers

The goal is to have a config file under `codegen.yml` and an npm script called `generator`.

> You can read more about GraphQL Code Generator [on its website](https://graphql-code-generator.com/docs/getting-started/).

[{]: <helper> (diffStep "2.1" module="server")

#### [Step 2.1: Install graphql-code-generator](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/03899a6)

##### Added codegen.yml
```diff
@@ -0,0 +1,11 @@
+┊  ┊ 1┊overwrite: true
+┊  ┊ 2┊schema: './schema/typeDefs.ts'
+┊  ┊ 3┊documents: null
+┊  ┊ 4┊require:
+┊  ┊ 5┊  - ts-node/register
+┊  ┊ 6┊generates:
+┊  ┊ 7┊  ./types.d.ts:
+┊  ┊ 8┊    plugins:
+┊  ┊ 9┊      - 'typescript-common'
+┊  ┊10┊      - 'typescript-server'
+┊  ┊11┊      - 'typescript-resolvers'
```

##### Changed package.json
```diff
@@ -4,7 +4,8 @@
 ┊ 4┊ 4┊  "private": true,
 ┊ 5┊ 5┊  "scripts": {
 ┊ 6┊ 6┊    "start": "npm run build:live",
-┊ 7┊  ┊    "build:live": "nodemon --exec ./node_modules/.bin/ts-node -- ./index.ts"
+┊  ┊ 7┊    "build:live": "nodemon --exec ./node_modules/.bin/ts-node -- ./index.ts",
+┊  ┊ 8┊    "generator": "gql-gen --config codegen.yml"
 ┊ 8┊ 9┊  },
 ┊ 9┊10┊  "devDependencies": {
 ┊10┊11┊    "@types/body-parser": "1.17.0",
```
```diff
@@ -14,7 +15,10 @@
 ┊14┊15┊    "@types/node": "10.11.3",
 ┊15┊16┊    "nodemon": "1.18.4",
 ┊16┊17┊    "ts-node": "7.0.1",
-┊17┊  ┊    "typescript": "3.1.1"
+┊  ┊18┊    "typescript": "3.1.1",
+┊  ┊19┊    "graphql-codegen-typescript-common": "0.16.0-alpha.207cae5d",
+┊  ┊20┊    "graphql-codegen-typescript-server": "0.16.0-alpha.207cae5d",
+┊  ┊21┊    "graphql-codegen-typescript-resolvers": "0.16.0-alpha.207cae5d"
 ┊18┊22┊  },
 ┊19┊23┊  "dependencies": {
 ┊20┊24┊    "apollo-server-express": "2.1.0",
```
```diff
@@ -22,6 +26,7 @@
 ┊22┊26┊    "cors": "2.8.4",
 ┊23┊27┊    "express": "4.16.3",
 ┊24┊28┊    "graphql": "14.0.2",
+┊  ┊29┊    "graphql-code-generator": "0.16.0-alpha.207cae5d",
 ┊25┊30┊    "moment": "2.22.1"
 ┊26┊31┊  }
-┊27┊  ┊}
+┊  ┊32┊}🚫↵
```

[}]: #

Few things here:

- `require: ts-node/register` - makes the ts-node compile TypeScript files
- `schema` - points to a file that exports the GraphQL Schema object or a string (it also accepts an url)
- `generates` - is an object where key is the filepath of an output
- `generates.plugins` - tells about the plugins we want to use

Now let's run the generator:

    yarn generator

Please note that the server doesn't have to be running in background because we import schema through a file.

Next, let's use the generated types:

[{]: <helper> (diffStep "2.3" module="server")

#### [Step 2.3: Use our types](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/5706af0)

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,5 +1,6 @@
 ┊1┊1┊import { IResolvers } from 'apollo-server-express';
 ┊2┊2┊import { Chat, db, Message, Recipient, User } from "../db";
+┊ ┊3┊import { ChatQueryArgs } from "../types";
 ┊3┊4┊
 ┊4┊5┊let users = db.users;
 ┊5┊6┊let chats = db.chats;
```
```diff
@@ -10,7 +11,7 @@
 ┊10┊11┊    // Show all users for the moment.
 ┊11┊12┊    users: (): User[] => users.filter(user => user.id !== currentUser),
 ┊12┊13┊    chats: (): Chat[] => chats.filter(chat => chat.listingMemberIds.includes(currentUser)),
-┊13┊  ┊    chat: (obj: any, {chatId}): Chat | null => chats.find(chat => chat.id === chatId) || null,
+┊  ┊14┊    chat: (obj: any, {chatId}: ChatQueryArgs): Chat | null => chats.find(chat => chat.id === Number(chatId)) || null,
 ┊14┊15┊  },
 ┊15┊16┊  Chat: {
 ┊16┊17┊    name: (chat: Chat): string => chat.name ? chat.name : users
```

[}]: #

Don't worry, they will be much more useful when we will write our first mutation.

### Generating types for client-side code

Let's do the same on the client:

    yarn add graphql-code-generator

and also to prepare everything:

    yarn gql-gen init

Exactly as with the server, you will need to answer few questions.

First, it will ask you what is the type of application you're going to build, pick "Vanilla JS application" (there's an Angular option but we will introduce it in the next chapter).
When it asks you for the schema, point it to our GraphQL Server `http://localhost:3000/graphql`.
Documents are available under `./src/graphql/**/*.ts`.
The output path should be: `./src/graphql.ts`

Plugins that you want to have selected are:

- TypeScript Common
- TypeScript Client

The goal again, is to have a config file under `codegen.yml` and an npm script called `generate`.

Please note that in this case, the server must be started before running the generator.

[{]: <helper> (diffStep "2.1" module="client")

#### [Step 2.1: Install graphql-code-generator](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/c09e60e)

##### Added codegen.yml
```diff
@@ -0,0 +1,8 @@
+┊ ┊1┊overwrite: true
+┊ ┊2┊schema: 'http://localhost:3000/graphql'
+┊ ┊3┊documents: './src/graphql/**/*.ts'
+┊ ┊4┊generates:
+┊ ┊5┊  ./src/graphql.ts:
+┊ ┊6┊    plugins:
+┊ ┊7┊      - 'typescript-common'
+┊ ┊8┊      - 'typescript-client'
```

##### Changed package.json
```diff
@@ -7,7 +7,8 @@
 ┊ 7┊ 7┊    "build": "ng build",
 ┊ 8┊ 8┊    "test": "ng test",
 ┊ 9┊ 9┊    "lint": "ng lint",
-┊10┊  ┊    "e2e": "ng e2e"
+┊  ┊10┊    "e2e": "ng e2e",
+┊  ┊11┊    "generator": "gql-gen --config codegen.yml"
 ┊11┊12┊  },
 ┊12┊13┊  "private": true,
 ┊13┊14┊  "dependencies": {
```
```diff
@@ -45,6 +46,7 @@
 ┊45┊46┊    "@types/jasminewd2": "2.0.3",
 ┊46┊47┊    "@types/node": "8.9.5",
 ┊47┊48┊    "codelyzer": "4.2.1",
+┊  ┊49┊    "graphql-code-generator": "0.16.0-alpha.207cae5d",
 ┊48┊50┊    "jasmine-core": "2.99.1",
 ┊49┊51┊    "jasmine-spec-reporter": "4.2.1",
 ┊50┊52┊    "karma": "1.7.1",
```
```diff
@@ -55,6 +57,8 @@
 ┊55┊57┊    "protractor": "5.3.1",
 ┊56┊58┊    "ts-node": "7.0.1",
 ┊57┊59┊    "tslint": "5.9.1",
-┊58┊  ┊    "typescript": "3.1.1"
+┊  ┊60┊    "typescript": "3.1.1",
+┊  ┊61┊    "graphql-codegen-typescript-common": "0.16.0-alpha.207cae5d",
+┊  ┊62┊    "graphql-codegen-typescript-client": "0.16.0-alpha.207cae5d"
 ┊59┊63┊  }
 ┊60┊64┊}
```

[}]: #

We saw how to use them on the server but let's see how easy it is to take advantage of them in Apollo.

First thing you should know is most of the methods of Apollo service accepts generic types.

#### What are Generic Types

A major part of software engineering is building components that not only have well-defined and consistent APIs, but are also reusable. Components that are capable of working on the data of today as well as the data of tomorrow will give you the most flexible capabilities for building up large software systems.

We like to work on an code samples so let's do some functional programming and create the `map` method that accepts a transform function and return a new result.

Without generics, we would have to use a specific type or `any`:

```typescript
function map(mapFn: (value: any) => any) {
  return function(source: any): any {
    return mapFn(source);
  };
}
```

Imagine, we want to use `map` function to pick user's id from an object:

```typescript
interface User {
  id: number;
}
const me = {
  id: 1,
};

const pickUserId = map(user => user.id);

console.log(pickUserId(me)); // outputs: 1
```

Great, it works, we get the expected result but it could be done way better.

What is the main issue here?
It accepts an object of any type and get `any` in return. It's not very useful because we know it return a `number` and receive a `User` but TypeScript doesn't know that and later on, in other part of the code, we end up with no information about the result.

That's where Generic Types jumps in!

With generics, it could look like this:

```typescript
function map<T, R>(mapFn: (value: T) => R) {
  return function(source: T): R {
    return mapFn(source);
  };
}
```

There's... a lot! So let's break it:

- `map<T, R>` - the map function accepts two generic types
- `(mapFn: (value: T) => R)` - the only argument is a function that accepts an object of type `T` and returns value of type `R`
- `function (source: T): R => {...}` - it's a higher order function that accepts a source of type `T` and transforms to be `R`.

Back to our example, now with generic types:

```typescript
interface User {
  id: number;
}
const me = {
  id: 1,
};

const pickUserId = map<User, number>(user => user.id);

console.log(pickUserId(me)); // outputs: 1
```

Our `map` and `pickUserId` function are strongly typed now so when you try to provide an object that has no `id` field or the field is a `string` you'll receive a proper information from TypeScript (and your IDE).

#### Make client-side code strongly typed

With all that knowledge let's use how to use those generated typed with `Apollo` service:

[{]: <helper> (diffStep "2.3" module="client")

#### [Step 2.3: Use the generated types](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/f5a1f23)

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;components&#x2F;chat-item&#x2F;chat-item.component.ts
```diff
@@ -1,4 +1,5 @@
 ┊1┊1┊import {Component, Input} from '@angular/core';
+┊ ┊2┊import {GetChats} from '../../../../graphql';
 ┊2┊3┊
 ┊3┊4┊@Component({
 ┊4┊5┊  selector: 'app-chat-item',
```
```diff
@@ -17,5 +18,5 @@
 ┊17┊18┊export class ChatItemComponent {
 ┊18┊19┊  // tslint:disable-next-line:no-input-rename
 ┊19┊20┊  @Input('item')
-┊20┊  ┊  chat: any;
+┊  ┊21┊  chat: GetChats.Chats;
 ┊21┊22┊}
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;components&#x2F;chats-list&#x2F;chats-list.component.ts
```diff
@@ -1,4 +1,5 @@
 ┊1┊1┊import {Component, Input} from '@angular/core';
+┊ ┊2┊import {GetChats} from '../../../../graphql';
 ┊2┊3┊
 ┊3┊4┊@Component({
 ┊4┊5┊  selector: 'app-chats-list',
```
```diff
@@ -14,7 +15,7 @@
 ┊14┊15┊export class ChatsListComponent {
 ┊15┊16┊  // tslint:disable-next-line:no-input-rename
 ┊16┊17┊  @Input('items')
-┊17┊  ┊  chats: any[];
+┊  ┊18┊  chats: GetChats.Chats[];
 ┊18┊19┊
 ┊19┊20┊  constructor() {}
 ┊20┊21┊}
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;containers&#x2F;chats&#x2F;chats.component.ts
```diff
@@ -1,6 +1,7 @@
 ┊1┊1┊import {Component, OnInit} from '@angular/core';
 ┊2┊2┊import {ChatsService} from '../../../services/chats.service';
 ┊3┊3┊import {Observable} from 'rxjs';
+┊ ┊4┊import {GetChats} from '../../../../graphql';
 ┊4┊5┊
 ┊5┊6┊@Component({
 ┊6┊7┊  template: `
```
```diff
@@ -35,7 +36,7 @@
 ┊35┊36┊  styleUrls: ['./chats.component.scss'],
 ┊36┊37┊})
 ┊37┊38┊export class ChatsComponent implements OnInit {
-┊38┊  ┊  chats$: Observable<any[]>;
+┊  ┊39┊  chats$: Observable<GetChats.Chats[]>;
 ┊39┊40┊
 ┊40┊41┊  constructor(private chatsService: ChatsService) {
 ┊41┊42┊  }
```

##### Changed src&#x2F;app&#x2F;services&#x2F;chats.service.ts
```diff
@@ -2,6 +2,7 @@
 ┊2┊2┊import {Apollo} from 'apollo-angular';
 ┊3┊3┊import {Injectable} from '@angular/core';
 ┊4┊4┊import {getChatsQuery} from '../../graphql/getChats.query';
+┊ ┊5┊import {GetChats} from '../../graphql';
 ┊5┊6┊
 ┊6┊7┊@Injectable()
 ┊7┊8┊export class ChatsService {
```
```diff
@@ -10,7 +11,7 @@
 ┊10┊11┊  constructor(private apollo: Apollo) {}
 ┊11┊12┊
 ┊12┊13┊  getChats() {
-┊13┊  ┊    const query = this.apollo.watchQuery<any>({
+┊  ┊14┊    const query = this.apollo.watchQuery<GetChats.Query>({
 ┊14┊15┊      query: getChatsQuery,
 ┊15┊16┊      variables: {
 ┊16┊17┊        amount: this.messagesAmount,
```

[}]: #

As you can probably tell by now, methods like `watchQuery` and `mutate` (and others) accepts two generic types, first one describes the result and the second one the variables.

```typescript
export class ChatService {
  constructor(private apollo: Apollo) {}

  getChat(chatId: string, amount: number): GetChat.Chat {
    return this.apollo
      .watchQuery<GetChat.Query, GetChat.Variables>({
        query: getChatQuery,
        variables: {
          chatId,
          amount,
        },
      })
      .pipe(map(result => result.data.chat));
  }
}
```

Few things here.

- An object of `GetChat.Query` shape lives under the `result.data` because `watchQuery` returns an object of type `ApolloQueryResult<T>` where `T` is our `GetChat.Query`.

```typescript
export type ApolloQueryResult<T> = {
  data: T;
  // ... other fields
};
```

You get the same result with Mutation or Subscription.

- Thanks to the second argument and `GetChat.Variables` we as well as TypeScript (and an IDE) know that `chatId` is a string and `amount` accepts only a number. Whenever we try to pass a value of different kind an Error will pop out.

### Take full advantage of Codegen and generate ready to use Services!

I've got a fantasctic news, you don't have to manually provide those generated types to each type Apollo service is used. Because GraphQL Documents are statically analyzable, we prepared a codegen template specific for Apollo Angular users.

First, let's install `graphql-codegen-typescript-apollo-angular` with the Apollo Angular plugin and update the list of plugins in `codegen.yml`

    yarn add -D graphql-codegen-typescript-apollo-angular

[{]: <helper> (diffStep "2.4" file="package.json" module="client")

#### [Step 2.4: Use auto-generated GQL services](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/7fa30eb)

##### Changed codegen.yml
```diff
@@ -6,3 +6,4 @@
 ┊6┊6┊    plugins:
 ┊7┊7┊      - 'typescript-common'
 ┊8┊8┊      - 'typescript-client'
+┊ ┊9┊      - 'typescript-apollo-angular'
```

##### Changed package.json
```diff
@@ -59,6 +59,7 @@
 ┊59┊59┊    "tslint": "5.9.1",
 ┊60┊60┊    "typescript": "3.1.1",
 ┊61┊61┊    "graphql-codegen-typescript-common": "0.16.0-alpha.207cae5d",
-┊62┊  ┊    "graphql-codegen-typescript-client": "0.16.0-alpha.207cae5d"
+┊  ┊62┊    "graphql-codegen-typescript-client": "0.16.0-alpha.207cae5d",
+┊  ┊63┊    "graphql-codegen-typescript-apollo-angular": "0.16.0-alpha.207cae5d"
 ┊63┊64┊  }
 ┊64┊65┊}
```

##### Changed src&#x2F;app&#x2F;services&#x2F;chats.service.ts
```diff
@@ -1,21 +1,18 @@
 ┊ 1┊ 1┊import {map} from 'rxjs/operators';
-┊ 2┊  ┊import {Apollo} from 'apollo-angular';
 ┊ 3┊ 2┊import {Injectable} from '@angular/core';
-┊ 4┊  ┊import {getChatsQuery} from '../../graphql/getChats.query';
-┊ 5┊  ┊import {GetChats} from '../../graphql';
+┊  ┊ 3┊import {GetChatsGQL} from '../../graphql';
 ┊ 6┊ 4┊
 ┊ 7┊ 5┊@Injectable()
 ┊ 8┊ 6┊export class ChatsService {
 ┊ 9┊ 7┊  messagesAmount = 3;
 ┊10┊ 8┊
-┊11┊  ┊  constructor(private apollo: Apollo) {}
+┊  ┊ 9┊  constructor(
+┊  ┊10┊    private getChatsGQL: GetChatsGQL
+┊  ┊11┊  ) {}
 ┊12┊12┊
 ┊13┊13┊  getChats() {
-┊14┊  ┊    const query = this.apollo.watchQuery<GetChats.Query>({
-┊15┊  ┊      query: getChatsQuery,
-┊16┊  ┊      variables: {
-┊17┊  ┊        amount: this.messagesAmount,
-┊18┊  ┊      },
+┊  ┊14┊    const query = this.getChatsGQL.watch({
+┊  ┊15┊      amount: this.messagesAmount,
 ┊19┊16┊    });
 ┊20┊17┊    const chats$ = query.valueChanges.pipe(
 ┊21┊18┊      map((result) => result.data.chats)
```

##### Changed src&#x2F;graphql.ts
```diff
@@ -132,3 +132,92 @@
 ┊132┊132┊    id: string;
 ┊133┊133┊  };
 ┊134┊134┊}
+┊   ┊135┊
+┊   ┊136┊// ====================================================
+┊   ┊137┊// START: Apollo Angular template
+┊   ┊138┊// ====================================================
+┊   ┊139┊
+┊   ┊140┊import { Injectable } from "@angular/core";
+┊   ┊141┊import * as Apollo from "apollo-angular";
+┊   ┊142┊
+┊   ┊143┊import gql from "graphql-tag";
+┊   ┊144┊
+┊   ┊145┊// ====================================================
+┊   ┊146┊// GraphQL Fragments
+┊   ┊147┊// ====================================================
+┊   ┊148┊
+┊   ┊149┊export const ChatWithoutMessagesFragment = gql`
+┊   ┊150┊  fragment ChatWithoutMessages on Chat {
+┊   ┊151┊    id
+┊   ┊152┊    name
+┊   ┊153┊    picture
+┊   ┊154┊    allTimeMembers {
+┊   ┊155┊      id
+┊   ┊156┊    }
+┊   ┊157┊    unreadMessages
+┊   ┊158┊    isGroup
+┊   ┊159┊  }
+┊   ┊160┊`;
+┊   ┊161┊
+┊   ┊162┊export const MessageFragment = gql`
+┊   ┊163┊  fragment Message on Message {
+┊   ┊164┊    id
+┊   ┊165┊    chat {
+┊   ┊166┊      id
+┊   ┊167┊    }
+┊   ┊168┊    sender {
+┊   ┊169┊      id
+┊   ┊170┊      name
+┊   ┊171┊    }
+┊   ┊172┊    content
+┊   ┊173┊    createdAt
+┊   ┊174┊    type
+┊   ┊175┊    recipients {
+┊   ┊176┊      user {
+┊   ┊177┊        id
+┊   ┊178┊      }
+┊   ┊179┊      message {
+┊   ┊180┊        id
+┊   ┊181┊        chat {
+┊   ┊182┊          id
+┊   ┊183┊        }
+┊   ┊184┊      }
+┊   ┊185┊      chat {
+┊   ┊186┊        id
+┊   ┊187┊      }
+┊   ┊188┊      receivedAt
+┊   ┊189┊      readAt
+┊   ┊190┊    }
+┊   ┊191┊    ownership
+┊   ┊192┊  }
+┊   ┊193┊`;
+┊   ┊194┊
+┊   ┊195┊// ====================================================
+┊   ┊196┊// Apollo Services
+┊   ┊197┊// ====================================================
+┊   ┊198┊
+┊   ┊199┊@Injectable({
+┊   ┊200┊  providedIn: "root"
+┊   ┊201┊})
+┊   ┊202┊export class GetChatsGQL extends Apollo.Query<
+┊   ┊203┊  GetChats.Query,
+┊   ┊204┊  GetChats.Variables
+┊   ┊205┊> {
+┊   ┊206┊  document: any = gql`
+┊   ┊207┊    query GetChats($amount: Int) {
+┊   ┊208┊      chats {
+┊   ┊209┊        ...ChatWithoutMessages
+┊   ┊210┊        messages(amount: $amount) {
+┊   ┊211┊          ...Message
+┊   ┊212┊        }
+┊   ┊213┊      }
+┊   ┊214┊    }
+┊   ┊215┊
+┊   ┊216┊    ${ChatWithoutMessagesFragment}
+┊   ┊217┊    ${MessageFragment}
+┊   ┊218┊  `;
+┊   ┊219┊}
+┊   ┊220┊
+┊   ┊221┊// ====================================================
+┊   ┊222┊// END: Apollo Angular template
+┊   ┊223┊// ====================================================
```

[}]: #

Then you need to add `typescript-apollo-angular` next to other plugins in `codegen.yml`.

The Apollo Angular template generates a ready to use in your component, strongly typed Angular services, for every defined query, mutation or subscription.

It's possible thanks to the new API of Apollo Angular. More on that in ["Query, Mutation, Subscription services"](http://apollographql.com/docs/angular/basics/services.html?_ga=2.227615197.1327014552.1538988114-793224955.1532981447) chapter of the documentation.

Given an example:

```graphql
query GetChats($amount: Int) {
  chats {
    ...ChatWithoutMessages
    messages(amount: $amount) {
      ...Message
    }
  }
}
```

The Apollo Angular template, after you run `yarn generate`, outputs a service called `GetChatsGQL`:

```typescript
import { GetChatsGQL, GetChats } from '../graphql';

export class AppComponent {
  constructor(private getChatsGQL: GetChatsGQL) {}

  getChats(): GetChats.Chats[] {
    return this.getChatsGQL
      .watch({
        amount: 10,
      })
      .valueChanges.pipe(map(result => result.data.chats));
  }
}
```

> Remember, every operation should has an unique name.

It might look a bit different then what you have already learnt but we promise, the API is even easier.

But first, let's dive into how those services look like under the hood.

```typescript
import { Query } from 'apollo-angular';

@Injectable({
  providedIn: 'root',
})
export class GetChatsGQL extends Query<GetChats.Query, GetChats.Variables> {
  document = gql`
    here goes the document
  `;
}
```

Okay, seems easy but what is the `Query` class, you might ask!

Apollo Angular exposes three classes for three kinds of the GraphQL operation: `Query`, `Mutation` and `Subscription`. Each of them has a different API.

- `Query` has `fetch` and `watch`. First one behave like `Apollo.query()`, second one like `Apollo.watchQuery()`
- `Mutation` has `mutate`
- `Subscription` has `subscribe`

Because the `document` is already defined in a class, they all accept two arguments (Subscription has third one). First argument defines variables, second shapes the options. Thats why we used `this.getChatsGQL.watch({ amount: 10 })`.

Let's stop talking about the API itself and see what benefits it brings us:

- **Less code to write** - no need to create a network call, no need to create Typescript typings, no need to create a dedicated Angular service
- **Strongly typed out of the box — all** types are being generated, no need to write any Typescript definitions and struggle to keep them updated
- _More pleasent API_ to work with
- **Full developer experience of tools and IDEs**  —  development time, autocomplete and error checking, not only across your frontend app but also with your API teams!
- **Tree-shakable** thanks to Angular 6

Most IDEs with the GraphQL support (built-in or thanks to extensions) fully handles `.graphql` files and helps you with features like auto-completion, validation but they strugle with `gql` tag. To fully enjoy GraphQL we highly recommend to use static `.graphql` files.

With all that knowledge, let's use GQL services in our application:

[{]: <helper> (diffStep "2.4" files="src/app/services/chats.service.ts" module="client")

#### [Step 2.4: Use auto-generated GQL services](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/7fa30eb)

##### Changed src&#x2F;app&#x2F;services&#x2F;chats.service.ts
```diff
@@ -1,21 +1,18 @@
 ┊ 1┊ 1┊import {map} from 'rxjs/operators';
-┊ 2┊  ┊import {Apollo} from 'apollo-angular';
 ┊ 3┊ 2┊import {Injectable} from '@angular/core';
-┊ 4┊  ┊import {getChatsQuery} from '../../graphql/getChats.query';
-┊ 5┊  ┊import {GetChats} from '../../graphql';
+┊  ┊ 3┊import {GetChatsGQL} from '../../graphql';
 ┊ 6┊ 4┊
 ┊ 7┊ 5┊@Injectable()
 ┊ 8┊ 6┊export class ChatsService {
 ┊ 9┊ 7┊  messagesAmount = 3;
 ┊10┊ 8┊
-┊11┊  ┊  constructor(private apollo: Apollo) {}
+┊  ┊ 9┊  constructor(
+┊  ┊10┊    private getChatsGQL: GetChatsGQL
+┊  ┊11┊  ) {}
 ┊12┊12┊
 ┊13┊13┊  getChats() {
-┊14┊  ┊    const query = this.apollo.watchQuery<GetChats.Query>({
-┊15┊  ┊      query: getChatsQuery,
-┊16┊  ┊      variables: {
-┊17┊  ┊        amount: this.messagesAmount,
-┊18┊  ┊      },
+┊  ┊14┊    const query = this.getChatsGQL.watch({
+┊  ┊15┊      amount: this.messagesAmount,
 ┊19┊16┊    });
 ┊20┊17┊    const chats$ = query.valueChanges.pipe(
 ┊21┊18┊      map((result) => result.data.chats)
```

[}]: #


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/tree/master@1.3.0/.tortilla/manuals/views/step5.md) | [Next Step >](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/tree/master@1.3.0/.tortilla/manuals/views/step7.md) |
|:--------------------------------|--------------------------------:|

[}]: #
