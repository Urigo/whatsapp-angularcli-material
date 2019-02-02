# Step 16: TypeORM with PostgreSQL

[//]: # (head-end)


## Server

First of all you will have to install PostgreSQL on your operating system. Since there so many options (different Linux distributions, MacOS X, Windows...) I will assume that you already know how to install a software in your OS and take that part for granted.

Then you will have to install a couple of packages:

    yarn add pg reflect-metadata typeorm
    yarn add -D @types/pg

We aren't going to use plain SQL, instead we will use an Object-relational mapping framework (ORM) called `TypeORM`.
`TypeORM` takes advantage of Typescript classes and type declarations in order to infer the db structure.

We will need to enable support for experimental decorators, emit type metadata for decorators and disable strict property initialization:

[{]: <helper> (diffStep "6.1" files="tsconfig.json" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Changed tsconfig.json
```diff
@@ -30,7 +30,7 @@
 ┊30┊30┊    // See https://github.com/DefinitelyTyped/DefinitelyTyped/issues/21359
 ┊31┊31┊    "strictFunctionTypes": false,             /* Enable strict checking of function types. */
 ┊32┊32┊    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
-┊33┊  ┊    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
+┊  ┊33┊    "strictPropertyInitialization": false,    /* Enable strict checking of property initialization in classes. */
 ┊34┊34┊    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
 ┊35┊35┊    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */
 ┊36┊36┊
```
```diff
@@ -48,7 +48,7 @@
 ┊48┊48┊    // "typeRoots": [],                       /* List of folders to include type definitions from. */
 ┊49┊49┊    // "types": [],                           /* Type declaration files to be included in compilation. */
 ┊50┊50┊    // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
-┊51┊  ┊    "esModuleInterop": true                   /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
+┊  ┊51┊    "esModuleInterop": true,                  /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
 ┊52┊52┊    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
 ┊53┊53┊
 ┊54┊54┊    /* Source Map Options */
```
```diff
@@ -58,7 +58,7 @@
 ┊58┊58┊    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */
 ┊59┊59┊
 ┊60┊60┊    /* Experimental Options */
-┊61┊  ┊    // "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
-┊62┊  ┊    // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */
+┊  ┊61┊    "experimentalDecorators": true,           /* Enables experimental support for ES7 decorators. */
+┊  ┊62┊    "emitDecoratorMetadata": true             /* Enables experimental support for emitting type metadata for decorators. */
 ┊63┊63┊  }
 ┊64┊64┊}
```

[}]: #

The next step is to create Entities. An Entity is a class that maps to a database table. You can create a entity by defining a new class and mark it with @Entity():

[{]: <helper> (diffStep "6.1" files="entity" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Added entity&#x2F;Chat.ts
```diff
@@ -0,0 +1,79 @@
+┊  ┊ 1┊import { Entity, Column, PrimaryGeneratedColumn, OneToMany, JoinTable, ManyToMany, ManyToOne } from "typeorm";
+┊  ┊ 2┊import { Message } from "./Message";
+┊  ┊ 3┊import { User } from "./User";
+┊  ┊ 4┊import { Recipient } from "./Recipient";
+┊  ┊ 5┊
+┊  ┊ 6┊interface ChatConstructor {
+┊  ┊ 7┊  name?: string;
+┊  ┊ 8┊  picture?: string;
+┊  ┊ 9┊  allTimeMembers?: User[];
+┊  ┊10┊  listingMembers?: User[];
+┊  ┊11┊  actualGroupMembers?: User[];
+┊  ┊12┊  admins?: User[];
+┊  ┊13┊  owner?: User;
+┊  ┊14┊  messages?: Message[];
+┊  ┊15┊}
+┊  ┊16┊
+┊  ┊17┊@Entity()
+┊  ┊18┊export class Chat {
+┊  ┊19┊  @PrimaryGeneratedColumn()
+┊  ┊20┊  id: number;
+┊  ┊21┊
+┊  ┊22┊  @Column({nullable: true})
+┊  ┊23┊  name: string;
+┊  ┊24┊
+┊  ┊25┊  @Column({nullable: true})
+┊  ┊26┊  picture: string;
+┊  ┊27┊
+┊  ┊28┊  @ManyToMany(type => User, user => user.allTimeMemberChats, {cascade: ["insert", "update"], eager: false})
+┊  ┊29┊  @JoinTable()
+┊  ┊30┊  allTimeMembers: User[];
+┊  ┊31┊
+┊  ┊32┊  @ManyToMany(type => User, user => user.listingMemberChats, {cascade: ["insert", "update"], eager: false})
+┊  ┊33┊  @JoinTable()
+┊  ┊34┊  listingMembers: User[];
+┊  ┊35┊
+┊  ┊36┊  @ManyToMany(type => User, user => user.actualGroupMemberChats, {cascade: ["insert", "update"], eager: false})
+┊  ┊37┊  @JoinTable()
+┊  ┊38┊  actualGroupMembers?: User[];
+┊  ┊39┊
+┊  ┊40┊  @ManyToMany(type => User, user => user.adminChats, {cascade: ["insert", "update"], eager: false})
+┊  ┊41┊  @JoinTable()
+┊  ┊42┊  admins?: User[];
+┊  ┊43┊
+┊  ┊44┊  @ManyToOne(type => User, user => user.ownerChats, {cascade: ["insert", "update"], eager: false})
+┊  ┊45┊  owner?: User | null;
+┊  ┊46┊
+┊  ┊47┊  @OneToMany(type => Message, message => message.chat, {cascade: ["insert", "update"], eager: true})
+┊  ┊48┊  messages: Message[];
+┊  ┊49┊
+┊  ┊50┊  @OneToMany(type => Recipient, recipient => recipient.chat)
+┊  ┊51┊  recipients: Recipient[];
+┊  ┊52┊
+┊  ┊53┊  constructor({name, picture, allTimeMembers, listingMembers, actualGroupMembers, admins, owner, messages}: ChatConstructor = {}) {
+┊  ┊54┊    if (name) {
+┊  ┊55┊      this.name = name;
+┊  ┊56┊    }
+┊  ┊57┊    if (picture) {
+┊  ┊58┊      this.picture = picture;
+┊  ┊59┊    }
+┊  ┊60┊    if (allTimeMembers) {
+┊  ┊61┊      this.allTimeMembers = allTimeMembers;
+┊  ┊62┊    }
+┊  ┊63┊    if (listingMembers) {
+┊  ┊64┊      this.listingMembers = listingMembers;
+┊  ┊65┊    }
+┊  ┊66┊    if (actualGroupMembers) {
+┊  ┊67┊      this.actualGroupMembers = actualGroupMembers;
+┊  ┊68┊    }
+┊  ┊69┊    if (admins) {
+┊  ┊70┊      this.admins = admins;
+┊  ┊71┊    }
+┊  ┊72┊    if (owner) {
+┊  ┊73┊      this.owner = owner;
+┊  ┊74┊    }
+┊  ┊75┊    if (messages) {
+┊  ┊76┊      this.messages = messages;
+┊  ┊77┊    }
+┊  ┊78┊  }
+┊  ┊79┊}
```

##### Added entity&#x2F;Message.ts
```diff
@@ -0,0 +1,70 @@
+┊  ┊ 1┊import {
+┊  ┊ 2┊  Entity, Column, PrimaryGeneratedColumn, OneToMany, ManyToOne, ManyToMany, JoinTable, CreateDateColumn
+┊  ┊ 3┊} from "typeorm";
+┊  ┊ 4┊import { Chat } from "./Chat";
+┊  ┊ 5┊import { User } from "./User";
+┊  ┊ 6┊import { Recipient } from "./Recipient";
+┊  ┊ 7┊import { MessageType } from "../db";
+┊  ┊ 8┊
+┊  ┊ 9┊interface MessageConstructor {
+┊  ┊10┊  sender?: User;
+┊  ┊11┊  content?: string;
+┊  ┊12┊  createdAt?: Date,
+┊  ┊13┊  type?: MessageType;
+┊  ┊14┊  recipients?: Recipient[];
+┊  ┊15┊  holders?: User[];
+┊  ┊16┊  chat?: Chat;
+┊  ┊17┊}
+┊  ┊18┊
+┊  ┊19┊@Entity()
+┊  ┊20┊export class Message {
+┊  ┊21┊  @PrimaryGeneratedColumn()
+┊  ┊22┊  id: number;
+┊  ┊23┊
+┊  ┊24┊  @ManyToOne(type => User, user => user.senderMessages, {eager: true})
+┊  ┊25┊  sender: User;
+┊  ┊26┊
+┊  ┊27┊  @Column()
+┊  ┊28┊  content: string;
+┊  ┊29┊
+┊  ┊30┊  @CreateDateColumn({nullable: true})
+┊  ┊31┊  createdAt: Date;
+┊  ┊32┊
+┊  ┊33┊  @Column()
+┊  ┊34┊  type: number;
+┊  ┊35┊
+┊  ┊36┊  @OneToMany(type => Recipient, recipient => recipient.message, {cascade: ["insert", "update"], eager: true})
+┊  ┊37┊  recipients: Recipient[];
+┊  ┊38┊
+┊  ┊39┊  @ManyToMany(type => User, user => user.holderMessages, {cascade: ["insert", "update"], eager: true})
+┊  ┊40┊  @JoinTable()
+┊  ┊41┊  holders: User[];
+┊  ┊42┊
+┊  ┊43┊  @ManyToOne(type => Chat, chat => chat.messages)
+┊  ┊44┊  chat: Chat;
+┊  ┊45┊
+┊  ┊46┊  constructor({sender, content, createdAt, type, recipients, holders, chat}: MessageConstructor = {}) {
+┊  ┊47┊    if (sender) {
+┊  ┊48┊      this.sender = sender;
+┊  ┊49┊    }
+┊  ┊50┊    if (content) {
+┊  ┊51┊      this.content = content;
+┊  ┊52┊    }
+┊  ┊53┊    if (createdAt) {
+┊  ┊54┊      this.createdAt = createdAt;
+┊  ┊55┊    }
+┊  ┊56┊    if (type) {
+┊  ┊57┊      this.type = type;
+┊  ┊58┊    }
+┊  ┊59┊    if (recipients) {
+┊  ┊60┊      recipients.forEach(recipient => recipient.message = this);
+┊  ┊61┊      this.recipients = recipients;
+┊  ┊62┊    }
+┊  ┊63┊    if (holders) {
+┊  ┊64┊      this.holders = holders;
+┊  ┊65┊    }
+┊  ┊66┊    if (chat) {
+┊  ┊67┊      this.chat = chat;
+┊  ┊68┊    }
+┊  ┊69┊  }
+┊  ┊70┊}
```

##### Added entity&#x2F;Recipient.ts
```diff
@@ -0,0 +1,44 @@
+┊  ┊ 1┊import { Entity, ManyToOne, Column } from "typeorm";
+┊  ┊ 2┊import { Message } from "./Message";
+┊  ┊ 3┊import { User } from "./User";
+┊  ┊ 4┊import { Chat } from "./Chat";
+┊  ┊ 5┊
+┊  ┊ 6┊interface RecipientConstructor {
+┊  ┊ 7┊  user?: User;
+┊  ┊ 8┊  message?: Message;
+┊  ┊ 9┊  receivedAt?: Date;
+┊  ┊10┊  readAt?: Date;
+┊  ┊11┊}
+┊  ┊12┊
+┊  ┊13┊@Entity()
+┊  ┊14┊export class Recipient {
+┊  ┊15┊  @ManyToOne(type => User, user => user.recipients, { primary: true })
+┊  ┊16┊  user: User;
+┊  ┊17┊
+┊  ┊18┊  @ManyToOne(type => Message, message => message.recipients, { primary: true })
+┊  ┊19┊  message: Message;
+┊  ┊20┊
+┊  ┊21┊  @ManyToOne(type => Chat, chat => chat.recipients)
+┊  ┊22┊  chat: Chat;
+┊  ┊23┊
+┊  ┊24┊  @Column({nullable: true})
+┊  ┊25┊  receivedAt: Date;
+┊  ┊26┊
+┊  ┊27┊  @Column({nullable: true})
+┊  ┊28┊  readAt: Date;
+┊  ┊29┊
+┊  ┊30┊  constructor({user, message, receivedAt, readAt}: RecipientConstructor = {}) {
+┊  ┊31┊    if (user) {
+┊  ┊32┊      this.user = user;
+┊  ┊33┊    }
+┊  ┊34┊    if (message) {
+┊  ┊35┊      this.message = message;
+┊  ┊36┊    }
+┊  ┊37┊    if (receivedAt) {
+┊  ┊38┊      this.receivedAt = receivedAt;
+┊  ┊39┊    }
+┊  ┊40┊    if (readAt) {
+┊  ┊41┊      this.readAt = readAt;
+┊  ┊42┊    }
+┊  ┊43┊  }
+┊  ┊44┊}
```

##### Added entity&#x2F;User.ts
```diff
@@ -0,0 +1,75 @@
+┊  ┊ 1┊import { Entity, Column, PrimaryGeneratedColumn, ManyToMany, OneToMany } from "typeorm";
+┊  ┊ 2┊import { Chat } from "./Chat";
+┊  ┊ 3┊import { Message } from "./Message";
+┊  ┊ 4┊import { Recipient } from "./Recipient";
+┊  ┊ 5┊
+┊  ┊ 6┊interface UserConstructor {
+┊  ┊ 7┊  username?: string;
+┊  ┊ 8┊  password?: string;
+┊  ┊ 9┊  name?: string;
+┊  ┊10┊  picture?: string;
+┊  ┊11┊  phone?: string;
+┊  ┊12┊}
+┊  ┊13┊
+┊  ┊14┊@Entity()
+┊  ┊15┊export class User {
+┊  ┊16┊  @PrimaryGeneratedColumn()
+┊  ┊17┊  id: number;
+┊  ┊18┊
+┊  ┊19┊  @Column()
+┊  ┊20┊  username: string;
+┊  ┊21┊
+┊  ┊22┊  @Column()
+┊  ┊23┊  password: string;
+┊  ┊24┊
+┊  ┊25┊  @Column()
+┊  ┊26┊  name: string;
+┊  ┊27┊
+┊  ┊28┊  @Column({nullable: true})
+┊  ┊29┊  picture: string;
+┊  ┊30┊
+┊  ┊31┊  @Column({nullable: true})
+┊  ┊32┊  phone?: string;
+┊  ┊33┊
+┊  ┊34┊  @ManyToMany(type => Chat, chat => chat.allTimeMembers)
+┊  ┊35┊  allTimeMemberChats: Chat[];
+┊  ┊36┊
+┊  ┊37┊  @ManyToMany(type => Chat, chat => chat.listingMembers)
+┊  ┊38┊  listingMemberChats: Chat[];
+┊  ┊39┊
+┊  ┊40┊  @ManyToMany(type => Chat, chat => chat.actualGroupMembers)
+┊  ┊41┊  actualGroupMemberChats: Chat[];
+┊  ┊42┊
+┊  ┊43┊  @ManyToMany(type => Chat, chat => chat.admins)
+┊  ┊44┊  adminChats: Chat[];
+┊  ┊45┊
+┊  ┊46┊  @ManyToMany(type => Message, message => message.holders)
+┊  ┊47┊  holderMessages: Message[];
+┊  ┊48┊
+┊  ┊49┊  @OneToMany(type => Chat, chat => chat.owner)
+┊  ┊50┊  ownerChats: Chat[];
+┊  ┊51┊
+┊  ┊52┊  @OneToMany(type => Message, message => message.sender)
+┊  ┊53┊  senderMessages: Message[];
+┊  ┊54┊
+┊  ┊55┊  @OneToMany(type => Recipient, recipient => recipient.user)
+┊  ┊56┊  recipients: Recipient[];
+┊  ┊57┊
+┊  ┊58┊  constructor({username, password, name, picture, phone}: UserConstructor = {}) {
+┊  ┊59┊    if (username) {
+┊  ┊60┊      this.username = username;
+┊  ┊61┊    }
+┊  ┊62┊    if (password) {
+┊  ┊63┊      this.password = password;
+┊  ┊64┊    }
+┊  ┊65┊    if (name) {
+┊  ┊66┊      this.name = name;
+┊  ┊67┊    }
+┊  ┊68┊    if (picture) {
+┊  ┊69┊      this.picture = picture;
+┊  ┊70┊    }
+┊  ┊71┊    if (phone) {
+┊  ┊72┊      this.phone = phone;
+┊  ┊73┊    }
+┊  ┊74┊  }
+┊  ┊75┊}
```

[}]: #

Basic entities consist of columns and relations. Each entity MUST have a primary column.

Each entity must be registered in your connection options:

[{]: <helper> (diffStep "6.1" files="ormconfig.json" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Added ormconfig.json
```diff
@@ -0,0 +1,24 @@
+┊  ┊ 1┊{
+┊  ┊ 2┊   "type": "postgres",
+┊  ┊ 3┊   "host": "localhost",
+┊  ┊ 4┊   "port": 5432,
+┊  ┊ 5┊   "username": "test",
+┊  ┊ 6┊   "password": "",
+┊  ┊ 7┊   "database": "test",
+┊  ┊ 8┊   "synchronize": true,
+┊  ┊ 9┊   "logging": false,
+┊  ┊10┊   "entities": [
+┊  ┊11┊      "entity/**/*.ts"
+┊  ┊12┊   ],
+┊  ┊13┊   "migrations": [
+┊  ┊14┊      "migration/**/*.ts"
+┊  ┊15┊   ],
+┊  ┊16┊   "subscribers": [
+┊  ┊17┊      "subscriber/**/*.ts"
+┊  ┊18┊   ],
+┊  ┊19┊   "cli": {
+┊  ┊20┊      "entitiesDir": "entity",
+┊  ┊21┊      "migrationsDir": "migration",
+┊  ┊22┊      "subscribersDir": "subscriber"
+┊  ┊23┊   }
+┊  ┊24┊}🚫↵
```

[}]: #

Since database table consist of columns your entities must consist of columns too. Each entity class property you marked with @Column will be mapped to a database table column.
Each entity must have at least one primary column. There are several types of primary columns, but in our case `@PrimaryGeneratedColumn()` creates a primary column which value will be automatically generated with an auto-increment value.
`@CreateDateColumn` is a special column that is automatically set to the entity's insertion date. You don't need set this column - it will be automatically set.
For the Recipient Entity we use a composite primary key that consists of two foreign keys.

The next thing to do is to create a connection with the database before firing up the web server:

[{]: <helper> (diffStep "6.1" files="index.ts" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Changed index.ts
```diff
@@ -1,3 +1,5 @@
+┊ ┊1┊// For TypeORM
+┊ ┊2┊import "reflect-metadata";
 ┊1┊3┊import { schema } from "./schema";
 ┊2┊4┊import bodyParser from "body-parser";
 ┊3┊5┊import cors from 'cors';
```
```diff
@@ -6,10 +8,10 @@
 ┊ 6┊ 8┊import passport from "passport";
 ┊ 7┊ 9┊import basicStrategy from 'passport-http';
 ┊ 8┊10┊import bcrypt from 'bcrypt-nodejs';
-┊ 9┊  ┊import { db, UserDb } from "./db";
 ┊10┊11┊import { createServer } from "http";
-┊11┊  ┊
-┊12┊  ┊let users = db.users;
+┊  ┊12┊import { createConnection } from "typeorm";
+┊  ┊13┊import { User } from "./entity/User";
+┊  ┊14┊import { addSampleData } from "./db";
 ┊13┊15┊
 ┊14┊16┊function generateHash(password: string) {
 ┊15┊17┊  return bcrypt.hashSync(password, bcrypt.genSaltSync(8));
```
```diff
@@ -19,93 +21,96 @@
 ┊ 19┊ 21┊  return bcrypt.compareSync(password, localPassword);
 ┊ 20┊ 22┊}
 ┊ 21┊ 23┊
-┊ 22┊   ┊passport.use('basic-signin', new basicStrategy.BasicStrategy(
-┊ 23┊   ┊  function (username: string, password: string, done: any) {
-┊ 24┊   ┊    const user = users.find(user => user.username == username);
-┊ 25┊   ┊    if (user && validPassword(password, user.password)) {
-┊ 26┊   ┊      return done(null, user);
+┊   ┊ 24┊createConnection().then(async connection => {
+┊   ┊ 25┊  await addSampleData(connection);
+┊   ┊ 26┊
+┊   ┊ 27┊  passport.use('basic-signin', new basicStrategy.BasicStrategy(
+┊   ┊ 28┊    async function (username: string, password: string, done: any) {
+┊   ┊ 29┊      const user = await connection.getRepository(User).findOne({where: { username }});
+┊   ┊ 30┊      if (user && validPassword(password, user.password)) {
+┊   ┊ 31┊        return done(null, user);
+┊   ┊ 32┊      }
+┊   ┊ 33┊      return done(null, false);
 ┊ 27┊ 34┊    }
-┊ 28┊   ┊    return done(null, false);
-┊ 29┊   ┊  }
-┊ 30┊   ┊));
-┊ 31┊   ┊
-┊ 32┊   ┊passport.use('basic-signup', new basicStrategy.BasicStrategy({passReqToCallback: true},
-┊ 33┊   ┊  function (req: any, username: any, password: any, done: any) {
-┊ 34┊   ┊    const userExists = !!users.find(user => user.username === username);
-┊ 35┊   ┊    if (!userExists && password && req.body.name) {
-┊ 36┊   ┊      const user: UserDb = {
-┊ 37┊   ┊        id: (users.length && users[users.length - 1].id + 1) || 1,
-┊ 38┊   ┊        username,
-┊ 39┊   ┊        password: generateHash(password),
-┊ 40┊   ┊        name: req.body.name,
-┊ 41┊   ┊      };
-┊ 42┊   ┊      users.push(user);
-┊ 43┊   ┊      return done(null, user);
+┊   ┊ 35┊  ));
+┊   ┊ 36┊
+┊   ┊ 37┊  passport.use('basic-signup', new basicStrategy.BasicStrategy({passReqToCallback: true},
+┊   ┊ 38┊    async function (req: any, username: string, password: string, done: any) {
+┊   ┊ 39┊      const userExists = !!(await connection.getRepository(User).findOne({where: { username }}));
+┊   ┊ 40┊      if (!userExists && password && req.body.name) {
+┊   ┊ 41┊        const user = await connection.manager.save(new User({
+┊   ┊ 42┊          username,
+┊   ┊ 43┊          password: generateHash(password),
+┊   ┊ 44┊          name: req.body.name,
+┊   ┊ 45┊        }));
+┊   ┊ 46┊        return done(null, user);
+┊   ┊ 47┊      }
+┊   ┊ 48┊      return done(null, false);
 ┊ 44┊ 49┊    }
-┊ 45┊   ┊    return done(null, false);
-┊ 46┊   ┊  }
-┊ 47┊   ┊));
+┊   ┊ 50┊  ));
 ┊ 48┊ 51┊
-┊ 49┊   ┊const PORT = 3000;
+┊   ┊ 52┊  const PORT = 3000;
 ┊ 50┊ 53┊
-┊ 51┊   ┊const app = express();
+┊   ┊ 54┊  const app = express();
 ┊ 52┊ 55┊
-┊ 53┊   ┊app.use(cors());
-┊ 54┊   ┊app.use(bodyParser.json());
-┊ 55┊   ┊app.use(passport.initialize());
+┊   ┊ 56┊  app.use(cors());
+┊   ┊ 57┊  app.use(bodyParser.json());
+┊   ┊ 58┊  app.use(passport.initialize());
 ┊ 56┊ 59┊
-┊ 57┊   ┊app.post('/signup',
-┊ 58┊   ┊  passport.authenticate('basic-signup', {session: false}),
-┊ 59┊   ┊  function (req: any, res) {
-┊ 60┊   ┊    res.json(req.user);
-┊ 61┊   ┊  });
+┊   ┊ 60┊  app.post('/signup',
+┊   ┊ 61┊    passport.authenticate('basic-signup', {session: false}),
+┊   ┊ 62┊    function (req: any, res) {
+┊   ┊ 63┊      res.json(req.user);
+┊   ┊ 64┊    });
 ┊ 62┊ 65┊
-┊ 63┊   ┊app.use(passport.authenticate('basic-signin', {session: false}));
+┊   ┊ 66┊  app.use(passport.authenticate('basic-signin', {session: false}));
 ┊ 64┊ 67┊
-┊ 65┊   ┊app.post('/signin', function (req: any, res) {
-┊ 66┊   ┊  res.json(req.user);
-┊ 67┊   ┊});
+┊   ┊ 68┊  app.post('/signin', function (req, res) {
+┊   ┊ 69┊    res.json(req.user);
+┊   ┊ 70┊  });
 ┊ 68┊ 71┊
-┊ 69┊   ┊const apollo = new ApolloServer({
-┊ 70┊   ┊  schema,
-┊ 71┊   ┊  context(received: any) {
-┊ 72┊   ┊    return {
-┊ 73┊   ┊      user: received.connection ? received.connection.context.user : received.req!['user'],
-┊ 74┊   ┊    }
-┊ 75┊   ┊  },
-┊ 76┊   ┊  subscriptions: {
-┊ 77┊   ┊    onConnect: (connectionParams: any, webSocket: any) => {
-┊ 78┊   ┊      if (connectionParams.authToken) {
-┊ 79┊   ┊        // create a buffer and tell it the data coming in is base64
-┊ 80┊   ┊        const buf = new Buffer(connectionParams.authToken.split(' ')[1], 'base64');
-┊ 81┊   ┊        // read it back out as a string
-┊ 82┊   ┊        const [username, password]: string[] = buf.toString().split(':');
-┊ 83┊   ┊        if (username && password) {
-┊ 84┊   ┊          const user = users.find(user => user.username == username);
-┊ 85┊   ┊
-┊ 86┊   ┊          if (user && validPassword(password, user.password)) {
-┊ 87┊   ┊            // Set context for the WebSocket
-┊ 88┊   ┊            return {user};
-┊ 89┊   ┊          } else {
-┊ 90┊   ┊            throw new Error('Wrong credentials!');
+┊   ┊ 72┊  const apollo = new ApolloServer({
+┊   ┊ 73┊    schema,
+┊   ┊ 74┊    context(received: any) {
+┊   ┊ 75┊      return {
+┊   ┊ 76┊        user: received.connection ? received.connection.context.user : received.req!['user'],
+┊   ┊ 77┊        connection,
+┊   ┊ 78┊      }
+┊   ┊ 79┊    },
+┊   ┊ 80┊    subscriptions: {
+┊   ┊ 81┊      onConnect: async (connectionParams: any, webSocket: any) => {
+┊   ┊ 82┊        if (connectionParams.authToken) {
+┊   ┊ 83┊          // Create a buffer and tell it the data coming in is base64
+┊   ┊ 84┊          const buf = new Buffer(connectionParams.authToken.split(' ')[1], 'base64');
+┊   ┊ 85┊          // Read it back out as a string
+┊   ┊ 86┊          const [username, password]: string[] = buf.toString().split(':');
+┊   ┊ 87┊          if (username && password) {
+┊   ┊ 88┊            const user = await connection.getRepository(User).findOne({where: { username }});
+┊   ┊ 89┊
+┊   ┊ 90┊            if (user && validPassword(password, user.password)) {
+┊   ┊ 91┊              // Set context for the WebSocket
+┊   ┊ 92┊              return {user};
+┊   ┊ 93┊            } else {
+┊   ┊ 94┊              throw new Error('Wrong credentials!');
+┊   ┊ 95┊            }
 ┊ 91┊ 96┊          }
 ┊ 92┊ 97┊        }
+┊   ┊ 98┊        throw new Error('Missing auth token!');
 ┊ 93┊ 99┊      }
-┊ 94┊   ┊      throw new Error('Missing auth token!');
 ┊ 95┊100┊    }
-┊ 96┊   ┊  }
-┊ 97┊   ┊});
+┊   ┊101┊  });
 ┊ 98┊102┊
-┊ 99┊   ┊apollo.applyMiddleware({
-┊100┊   ┊  app,
-┊101┊   ┊  path: '/graphql'
-┊102┊   ┊});
+┊   ┊103┊  apollo.applyMiddleware({
+┊   ┊104┊    app,
+┊   ┊105┊    path: '/graphql'
+┊   ┊106┊  });
 ┊103┊107┊
-┊104┊   ┊// Wrap the Express server
-┊105┊   ┊const ws = createServer(app);
+┊   ┊108┊  // Wrap the Express server
+┊   ┊109┊  const ws = createServer(app);
 ┊106┊110┊
-┊107┊   ┊apollo.installSubscriptionHandlers(ws);
+┊   ┊111┊  apollo.installSubscriptionHandlers(ws);
 ┊108┊112┊
-┊109┊   ┊ws.listen(PORT, () => {
-┊110┊   ┊  console.log(`Apollo Server is now running on http://localhost:${PORT}`);
+┊   ┊113┊  ws.listen(PORT, () => {
+┊   ┊114┊    console.log(`Apollo Server is now running on http://localhost:${PORT}`);
+┊   ┊115┊  });
 ┊111┊116┊});
```

[}]: #

We will also remove our fake db and replace it with some real data:

[{]: <helper> (diffStep "6.1" files="db.ts" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Changed db.ts
```diff
@@ -1,4 +1,11 @@
+┊  ┊ 1┊// For TypeORM
+┊  ┊ 2┊import "reflect-metadata";
+┊  ┊ 3┊import { Chat } from "./entity/Chat";
+┊  ┊ 4┊import { Recipient } from "./entity/Recipient";
 ┊ 1┊ 5┊import moment from 'moment';
+┊  ┊ 6┊import { Message } from "./entity/Message";
+┊  ┊ 7┊import { User } from "./entity/User";
+┊  ┊ 8┊import { Connection } from "typeorm";
 ┊ 2┊ 9┊
 ┊ 3┊10┊export enum MessageType {
 ┊ 4┊11┊  PICTURE,
```
```diff
@@ -6,433 +13,280 @@
 ┊  6┊ 13┊  LOCATION,
 ┊  7┊ 14┊}
 ┊  8┊ 15┊
-┊  9┊   ┊export interface UserDb {
-┊ 10┊   ┊  id: number,
-┊ 11┊   ┊  username: string,
-┊ 12┊   ┊  password: string,
-┊ 13┊   ┊  name: string,
-┊ 14┊   ┊  picture?: string | null,
-┊ 15┊   ┊  phone?: string | null,
-┊ 16┊   ┊}
-┊ 17┊   ┊
-┊ 18┊   ┊export interface ChatDb {
-┊ 19┊   ┊  id: number,
-┊ 20┊   ┊  name?: string | null,
-┊ 21┊   ┊  picture?: string | null,
-┊ 22┊   ┊  // All members, current and past ones.
-┊ 23┊   ┊  allTimeMemberIds: number[],
-┊ 24┊   ┊  // Whoever gets the chat listed. For groups includes past members who still didn't delete the group.
-┊ 25┊   ┊  listingMemberIds: number[],
-┊ 26┊   ┊  // Actual members of the group (they are not the only ones who get the group listed). Null for chats.
-┊ 27┊   ┊  actualGroupMemberIds?: number[] | null,
-┊ 28┊   ┊  adminIds?: number[] | null,
-┊ 29┊   ┊  ownerId?: number | null,
-┊ 30┊   ┊  messages: MessageDb[],
-┊ 31┊   ┊}
-┊ 32┊   ┊
-┊ 33┊   ┊export interface MessageDb {
-┊ 34┊   ┊  id: number,
-┊ 35┊   ┊  chatId: number,
-┊ 36┊   ┊  senderId: number,
-┊ 37┊   ┊  content: string,
-┊ 38┊   ┊  createdAt: number,
-┊ 39┊   ┊  type: MessageType,
-┊ 40┊   ┊  recipients: RecipientDb[],
-┊ 41┊   ┊  holderIds: number[],
-┊ 42┊   ┊}
-┊ 43┊   ┊
-┊ 44┊   ┊export interface RecipientDb {
-┊ 45┊   ┊  userId: number,
-┊ 46┊   ┊  messageId: number,
-┊ 47┊   ┊  chatId: number,
-┊ 48┊   ┊  receivedAt: number | null,
-┊ 49┊   ┊  readAt: number | null,
-┊ 50┊   ┊}
-┊ 51┊   ┊
-┊ 52┊   ┊const users: UserDb[] = [
-┊ 53┊   ┊  {
-┊ 54┊   ┊    id: 1,
+┊   ┊ 16┊export async function addSampleData(connection: Connection) {
+┊   ┊ 17┊  const user1 = new User({
 ┊ 55┊ 18┊    username: 'ethan',
 ┊ 56┊ 19┊    password: '$2a$08$NO9tkFLCoSqX1c5wk3s7z.JfxaVMKA.m7zUDdDwEquo4rvzimQeJm', // 111
 ┊ 57┊ 20┊    name: 'Ethan Gonzalez',
 ┊ 58┊ 21┊    picture: 'https://randomuser.me/api/portraits/thumb/men/1.jpg',
 ┊ 59┊ 22┊    phone: '+391234567890',
-┊ 60┊   ┊  },
-┊ 61┊   ┊  {
-┊ 62┊   ┊    id: 2,
+┊   ┊ 23┊  });
+┊   ┊ 24┊  await connection.manager.save(user1);
+┊   ┊ 25┊
+┊   ┊ 26┊  const user2 = new User({
 ┊ 63┊ 27┊    username: 'bryan',
 ┊ 64┊ 28┊    password: '$2a$08$xE4FuCi/ifxjL2S8CzKAmuKLwv18ktksSN.F3XYEnpmcKtpbpeZgO', // 222
 ┊ 65┊ 29┊    name: 'Bryan Wallace',
 ┊ 66┊ 30┊    picture: 'https://randomuser.me/api/portraits/thumb/men/2.jpg',
 ┊ 67┊ 31┊    phone: '+391234567891',
-┊ 68┊   ┊  },
-┊ 69┊   ┊  {
-┊ 70┊   ┊    id: 3,
+┊   ┊ 32┊  });
+┊   ┊ 33┊  await connection.manager.save(user2);
+┊   ┊ 34┊
+┊   ┊ 35┊  const user3 = new User({
 ┊ 71┊ 36┊    username: 'avery',
 ┊ 72┊ 37┊    password: '$2a$08$UHgH7J8G6z1mGQn2qx2kdeWv0jvgHItyAsL9hpEUI3KJmhVW5Q1d.', // 333
 ┊ 73┊ 38┊    name: 'Avery Stewart',
 ┊ 74┊ 39┊    picture: 'https://randomuser.me/api/portraits/thumb/women/1.jpg',
 ┊ 75┊ 40┊    phone: '+391234567892',
-┊ 76┊   ┊  },
-┊ 77┊   ┊  {
-┊ 78┊   ┊    id: 4,
+┊   ┊ 41┊  });
+┊   ┊ 42┊  await connection.manager.save(user3);
+┊   ┊ 43┊
+┊   ┊ 44┊  const user4 = new User({
 ┊ 79┊ 45┊    username: 'katie',
 ┊ 80┊ 46┊    password: '$2a$08$wR1k5Q3T9FC7fUgB7Gdb9Os/GV7dGBBf4PLlWT7HERMFhmFDt47xi', // 444
 ┊ 81┊ 47┊    name: 'Katie Peterson',
 ┊ 82┊ 48┊    picture: 'https://randomuser.me/api/portraits/thumb/women/2.jpg',
 ┊ 83┊ 49┊    phone: '+391234567893',
-┊ 84┊   ┊  },
-┊ 85┊   ┊  {
-┊ 86┊   ┊    id: 5,
+┊   ┊ 50┊  });
+┊   ┊ 51┊  await connection.manager.save(user4);
+┊   ┊ 52┊
+┊   ┊ 53┊  const user5 = new User({
 ┊ 87┊ 54┊    username: 'ray',
 ┊ 88┊ 55┊    password: '$2a$08$6.mbXqsDX82ZZ7q5d8Osb..JrGSsNp4R3IKj7mxgF6YGT0OmMw242', // 555
 ┊ 89┊ 56┊    name: 'Ray Edwards',
 ┊ 90┊ 57┊    picture: 'https://randomuser.me/api/portraits/thumb/men/3.jpg',
 ┊ 91┊ 58┊    phone: '+391234567894',
-┊ 92┊   ┊  },
-┊ 93┊   ┊  {
-┊ 94┊   ┊    id: 6,
+┊   ┊ 59┊  });
+┊   ┊ 60┊  await connection.manager.save(user5);
+┊   ┊ 61┊
+┊   ┊ 62┊  const user6 = new User({
 ┊ 95┊ 63┊    username: 'niko',
 ┊ 96┊ 64┊    password: '$2a$08$fL5lZR.Rwf9FWWe8XwwlceiPBBim8n9aFtaem.INQhiKT4.Ux3Uq.', // 666
 ┊ 97┊ 65┊    name: 'Niccolò Belli',
 ┊ 98┊ 66┊    picture: 'https://randomuser.me/api/portraits/thumb/men/4.jpg',
 ┊ 99┊ 67┊    phone: '+391234567895',
-┊100┊   ┊  },
-┊101┊   ┊  {
-┊102┊   ┊    id: 7,
+┊   ┊ 68┊  });
+┊   ┊ 69┊  await connection.manager.save(user6);
+┊   ┊ 70┊
+┊   ┊ 71┊  const user7 = new User({
 ┊103┊ 72┊    username: 'mario',
 ┊104┊ 73┊    password: '$2a$08$nDHDmWcVxDnH5DDT3HMMC.psqcnu6wBiOgkmJUy9IH..qxa3R6YrO', // 777
 ┊105┊ 74┊    name: 'Mario Rossi',
 ┊106┊ 75┊    picture: 'https://randomuser.me/api/portraits/thumb/men/5.jpg',
 ┊107┊ 76┊    phone: '+391234567896',
-┊108┊   ┊  },
-┊109┊   ┊];
+┊   ┊ 77┊  });
+┊   ┊ 78┊  await connection.manager.save(user7);
+┊   ┊ 79┊
+┊   ┊ 80┊
 ┊110┊ 81┊
-┊111┊   ┊const chats: ChatDb[] = [
-┊112┊   ┊  {
-┊113┊   ┊    id: 1,
-┊114┊   ┊    name: null,
-┊115┊   ┊    picture: null,
-┊116┊   ┊    allTimeMemberIds: [1, 3],
-┊117┊   ┊    listingMemberIds: [1, 3],
-┊118┊   ┊    adminIds: null,
-┊119┊   ┊    ownerId: null,
+┊   ┊ 82┊
+┊   ┊ 83┊  await connection.manager.save(new Chat({
+┊   ┊ 84┊    allTimeMembers: [user1, user3],
+┊   ┊ 85┊    listingMembers: [user1, user3],
 ┊120┊ 86┊    messages: [
-┊121┊   ┊      {
-┊122┊   ┊        id: 1,
-┊123┊   ┊        chatId: 1,
-┊124┊   ┊        senderId: 1,
+┊   ┊ 87┊      new Message({
+┊   ┊ 88┊        sender: user1,
 ┊125┊ 89┊        content: 'You on your way?',
-┊126┊   ┊        createdAt: moment().subtract(1, 'hours').unix(),
+┊   ┊ 90┊        createdAt: moment().subtract(1, 'hours').toDate(),
 ┊127┊ 91┊        type: MessageType.TEXT,
+┊   ┊ 92┊        holders: [user1, user3],
 ┊128┊ 93┊        recipients: [
-┊129┊   ┊          {
-┊130┊   ┊            userId: 3,
-┊131┊   ┊            messageId: 1,
-┊132┊   ┊            chatId: 1,
-┊133┊   ┊            receivedAt: null,
-┊134┊   ┊            readAt: null,
-┊135┊   ┊          },
+┊   ┊ 94┊          new Recipient({
+┊   ┊ 95┊            user: user3,
+┊   ┊ 96┊          }),
 ┊136┊ 97┊        ],
-┊137┊   ┊        holderIds: [1, 3],
-┊138┊   ┊      },
-┊139┊   ┊      {
-┊140┊   ┊        id: 2,
-┊141┊   ┊        chatId: 1,
-┊142┊   ┊        senderId: 3,
+┊   ┊ 98┊      }),
+┊   ┊ 99┊      new Message({
+┊   ┊100┊        sender: user3,
 ┊143┊101┊        content: 'Yep!',
-┊144┊   ┊        createdAt: moment().subtract(1, 'hours').add(5, 'minutes').unix(),
+┊   ┊102┊        createdAt: moment().subtract(1, 'hours').add(5, 'minutes').toDate(),
 ┊145┊103┊        type: MessageType.TEXT,
+┊   ┊104┊        holders: [user1, user3],
 ┊146┊105┊        recipients: [
-┊147┊   ┊          {
-┊148┊   ┊            userId: 1,
-┊149┊   ┊            messageId: 2,
-┊150┊   ┊            chatId: 1,
-┊151┊   ┊            receivedAt: null,
-┊152┊   ┊            readAt: null,
-┊153┊   ┊          },
+┊   ┊106┊          new Recipient({
+┊   ┊107┊            user: user1,
+┊   ┊108┊          }),
 ┊154┊109┊        ],
-┊155┊   ┊        holderIds: [3, 1],
-┊156┊   ┊      },
+┊   ┊110┊      }),
 ┊157┊111┊    ],
-┊158┊   ┊  },
-┊159┊   ┊  {
-┊160┊   ┊    id: 2,
-┊161┊   ┊    name: null,
-┊162┊   ┊    picture: null,
-┊163┊   ┊    allTimeMemberIds: [1, 4],
-┊164┊   ┊    listingMemberIds: [1, 4],
-┊165┊   ┊    adminIds: null,
-┊166┊   ┊    ownerId: null,
+┊   ┊112┊  }));
+┊   ┊113┊
+┊   ┊114┊  await connection.manager.save(new Chat({
+┊   ┊115┊    allTimeMembers: [user1, user4],
+┊   ┊116┊    listingMembers: [user1, user4],
 ┊167┊117┊    messages: [
-┊168┊   ┊      {
-┊169┊   ┊        id: 1,
-┊170┊   ┊        chatId: 2,
-┊171┊   ┊        senderId: 1,
+┊   ┊118┊      new Message({
+┊   ┊119┊        sender: user1,
 ┊172┊120┊        content: 'Hey, it\'s me',
-┊173┊   ┊        createdAt: moment().subtract(2, 'hours').unix(),
+┊   ┊121┊        createdAt: moment().subtract(2, 'hours').toDate(),
 ┊174┊122┊        type: MessageType.TEXT,
+┊   ┊123┊        holders: [user1, user4],
 ┊175┊124┊        recipients: [
-┊176┊   ┊          {
-┊177┊   ┊            userId: 4,
-┊178┊   ┊            messageId: 1,
-┊179┊   ┊            chatId: 2,
-┊180┊   ┊            receivedAt: null,
-┊181┊   ┊            readAt: null,
-┊182┊   ┊          },
+┊   ┊125┊          new Recipient({
+┊   ┊126┊            user: user4,
+┊   ┊127┊          }),
 ┊183┊128┊        ],
-┊184┊   ┊        holderIds: [1, 4],
-┊185┊   ┊      },
+┊   ┊129┊      }),
 ┊186┊130┊    ],
-┊187┊   ┊  },
-┊188┊   ┊  {
-┊189┊   ┊    id: 3,
-┊190┊   ┊    name: null,
-┊191┊   ┊    picture: null,
-┊192┊   ┊    allTimeMemberIds: [1, 5],
-┊193┊   ┊    listingMemberIds: [1, 5],
-┊194┊   ┊    adminIds: null,
-┊195┊   ┊    ownerId: null,
+┊   ┊131┊  }));
+┊   ┊132┊
+┊   ┊133┊  await connection.manager.save(new Chat({
+┊   ┊134┊    allTimeMembers: [user1, user5],
+┊   ┊135┊    listingMembers: [user1, user5],
 ┊196┊136┊    messages: [
-┊197┊   ┊      {
-┊198┊   ┊        id: 1,
-┊199┊   ┊        chatId: 3,
-┊200┊   ┊        senderId: 1,
+┊   ┊137┊      new Message({
+┊   ┊138┊        sender: user1,
 ┊201┊139┊        content: 'I should buy a boat',
-┊202┊   ┊        createdAt: moment().subtract(1, 'days').unix(),
+┊   ┊140┊        createdAt: moment().subtract(1, 'days').toDate(),
 ┊203┊141┊        type: MessageType.TEXT,
+┊   ┊142┊        holders: [user1, user5],
 ┊204┊143┊        recipients: [
-┊205┊   ┊          {
-┊206┊   ┊            userId: 5,
-┊207┊   ┊            messageId: 1,
-┊208┊   ┊            chatId: 3,
-┊209┊   ┊            receivedAt: null,
-┊210┊   ┊            readAt: null,
-┊211┊   ┊          },
+┊   ┊144┊          new Recipient({
+┊   ┊145┊            user: user5,
+┊   ┊146┊          }),
 ┊212┊147┊        ],
-┊213┊   ┊        holderIds: [1, 5],
-┊214┊   ┊      },
-┊215┊   ┊      {
-┊216┊   ┊        id: 2,
-┊217┊   ┊        chatId: 3,
-┊218┊   ┊        senderId: 1,
+┊   ┊148┊      }),
+┊   ┊149┊      new Message({
+┊   ┊150┊        sender: user1,
 ┊219┊151┊        content: 'You still there?',
-┊220┊   ┊        createdAt: moment().subtract(1, 'days').add(16, 'hours').unix(),
+┊   ┊152┊        createdAt: moment().subtract(1, 'days').add(16, 'hours').toDate(),
 ┊221┊153┊        type: MessageType.TEXT,
+┊   ┊154┊        holders: [user1, user5],
 ┊222┊155┊        recipients: [
-┊223┊   ┊          {
-┊224┊   ┊            userId: 5,
-┊225┊   ┊            messageId: 2,
-┊226┊   ┊            chatId: 3,
-┊227┊   ┊            receivedAt: null,
-┊228┊   ┊            readAt: null,
-┊229┊   ┊          },
+┊   ┊156┊          new Recipient({
+┊   ┊157┊            user: user5,
+┊   ┊158┊          }),
 ┊230┊159┊        ],
-┊231┊   ┊        holderIds: [1, 5],
-┊232┊   ┊      },
+┊   ┊160┊      }),
 ┊233┊161┊    ],
-┊234┊   ┊  },
-┊235┊   ┊  {
-┊236┊   ┊    id: 4,
-┊237┊   ┊    name: null,
-┊238┊   ┊    picture: null,
-┊239┊   ┊    allTimeMemberIds: [3, 4],
-┊240┊   ┊    listingMemberIds: [3, 4],
-┊241┊   ┊    adminIds: null,
-┊242┊   ┊    ownerId: null,
+┊   ┊162┊  }));
+┊   ┊163┊
+┊   ┊164┊  await connection.manager.save(new Chat({
+┊   ┊165┊    allTimeMembers: [user3, user4],
+┊   ┊166┊    listingMembers: [user3, user4],
 ┊243┊167┊    messages: [
-┊244┊   ┊      {
-┊245┊   ┊        id: 1,
-┊246┊   ┊        chatId: 4,
-┊247┊   ┊        senderId: 3,
+┊   ┊168┊      new Message({
+┊   ┊169┊        sender: user3,
 ┊248┊170┊        content: 'Look at my mukluks!',
-┊249┊   ┊        createdAt: moment().subtract(4, 'days').unix(),
+┊   ┊171┊        createdAt: moment().subtract(4, 'days').toDate(),
 ┊250┊172┊        type: MessageType.TEXT,
+┊   ┊173┊        holders: [user3, user4],
 ┊251┊174┊        recipients: [
-┊252┊   ┊          {
-┊253┊   ┊            userId: 4,
-┊254┊   ┊            messageId: 1,
-┊255┊   ┊            chatId: 4,
-┊256┊   ┊            receivedAt: null,
-┊257┊   ┊            readAt: null,
-┊258┊   ┊          },
+┊   ┊175┊          new Recipient({
+┊   ┊176┊            user: user4,
+┊   ┊177┊          }),
 ┊259┊178┊        ],
-┊260┊   ┊        holderIds: [3, 4],
-┊261┊   ┊      },
+┊   ┊179┊      }),
 ┊262┊180┊    ],
-┊263┊   ┊  },
-┊264┊   ┊  {
-┊265┊   ┊    id: 5,
-┊266┊   ┊    name: null,
-┊267┊   ┊    picture: null,
-┊268┊   ┊    allTimeMemberIds: [2, 5],
-┊269┊   ┊    listingMemberIds: [2, 5],
-┊270┊   ┊    adminIds: null,
-┊271┊   ┊    ownerId: null,
+┊   ┊181┊  }));
+┊   ┊182┊
+┊   ┊183┊  await connection.manager.save(new Chat({
+┊   ┊184┊    allTimeMembers: [user2, user5],
+┊   ┊185┊    listingMembers: [user2, user5],
 ┊272┊186┊    messages: [
-┊273┊   ┊      {
-┊274┊   ┊        id: 1,
-┊275┊   ┊        chatId: 5,
-┊276┊   ┊        senderId: 2,
+┊   ┊187┊      new Message({
+┊   ┊188┊        sender: user2,
 ┊277┊189┊        content: 'This is wicked good ice cream.',
-┊278┊   ┊        createdAt: moment().subtract(2, 'weeks').unix(),
+┊   ┊190┊        createdAt: moment().subtract(2, 'weeks').toDate(),
 ┊279┊191┊        type: MessageType.TEXT,
+┊   ┊192┊        holders: [user2, user5],
 ┊280┊193┊        recipients: [
-┊281┊   ┊          {
-┊282┊   ┊            userId: 5,
-┊283┊   ┊            messageId: 1,
-┊284┊   ┊            chatId: 5,
-┊285┊   ┊            receivedAt: null,
-┊286┊   ┊            readAt: null,
-┊287┊   ┊          },
+┊   ┊194┊          new Recipient({
+┊   ┊195┊            user: user5,
+┊   ┊196┊          }),
 ┊288┊197┊        ],
-┊289┊   ┊        holderIds: [2, 5],
-┊290┊   ┊      },
-┊291┊   ┊      {
-┊292┊   ┊        id: 2,
-┊293┊   ┊        chatId: 6,
-┊294┊   ┊        senderId: 5,
+┊   ┊198┊      }),
+┊   ┊199┊      new Message({
+┊   ┊200┊        sender: user5,
 ┊295┊201┊        content: 'Love it!',
-┊296┊   ┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').unix(),
+┊   ┊202┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').toDate(),
 ┊297┊203┊        type: MessageType.TEXT,
+┊   ┊204┊        holders: [user2, user5],
 ┊298┊205┊        recipients: [
-┊299┊   ┊          {
-┊300┊   ┊            userId: 2,
-┊301┊   ┊            messageId: 2,
-┊302┊   ┊            chatId: 5,
-┊303┊   ┊            receivedAt: null,
-┊304┊   ┊            readAt: null,
-┊305┊   ┊          },
+┊   ┊206┊          new Recipient({
+┊   ┊207┊            user: user2,
+┊   ┊208┊          }),
 ┊306┊209┊        ],
-┊307┊   ┊        holderIds: [5, 2],
-┊308┊   ┊      },
+┊   ┊210┊      }),
 ┊309┊211┊    ],
-┊310┊   ┊  },
-┊311┊   ┊  {
-┊312┊   ┊    id: 6,
-┊313┊   ┊    name: null,
-┊314┊   ┊    picture: null,
-┊315┊   ┊    allTimeMemberIds: [1, 6],
-┊316┊   ┊    listingMemberIds: [1],
-┊317┊   ┊    adminIds: null,
-┊318┊   ┊    ownerId: null,
-┊319┊   ┊    messages: [],
-┊320┊   ┊  },
-┊321┊   ┊  {
-┊322┊   ┊    id: 7,
-┊323┊   ┊    name: null,
-┊324┊   ┊    picture: null,
-┊325┊   ┊    allTimeMemberIds: [2, 1],
-┊326┊   ┊    listingMemberIds: [2],
-┊327┊   ┊    adminIds: null,
-┊328┊   ┊    ownerId: null,
-┊329┊   ┊    messages: [],
-┊330┊   ┊  },
-┊331┊   ┊  {
-┊332┊   ┊    id: 8,
-┊333┊   ┊    name: 'A user 0 group',
+┊   ┊212┊  }));
+┊   ┊213┊
+┊   ┊214┊  await connection.manager.save(new Chat({
+┊   ┊215┊    allTimeMembers: [user1, user6],
+┊   ┊216┊    listingMembers: [user1],
+┊   ┊217┊  }));
+┊   ┊218┊
+┊   ┊219┊  await connection.manager.save(new Chat({
+┊   ┊220┊    allTimeMembers: [user2, user1],
+┊   ┊221┊    listingMembers: [user2],
+┊   ┊222┊  }));
+┊   ┊223┊
+┊   ┊224┊  await connection.manager.save(new Chat({
+┊   ┊225┊    name: 'Ethan\'s group',
 ┊334┊226┊    picture: 'https://randomuser.me/api/portraits/thumb/lego/1.jpg',
-┊335┊   ┊    allTimeMemberIds: [1, 3, 4, 6],
-┊336┊   ┊    listingMemberIds: [1, 3, 4, 6],
-┊337┊   ┊    actualGroupMemberIds: [1, 4, 6],
-┊338┊   ┊    adminIds: [1, 6],
-┊339┊   ┊    ownerId: 1,
+┊   ┊227┊    allTimeMembers: [user1, user3, user4, user6],
+┊   ┊228┊    listingMembers: [user1, user3, user4, user6],
+┊   ┊229┊    actualGroupMembers: [user1, user4, user6],
+┊   ┊230┊    admins: [user1, user6],
+┊   ┊231┊    owner: user1,
 ┊340┊232┊    messages: [
-┊341┊   ┊      {
-┊342┊   ┊        id: 1,
-┊343┊   ┊        chatId: 8,
-┊344┊   ┊        senderId: 1,
+┊   ┊233┊      new Message({
+┊   ┊234┊        sender: user1,
 ┊345┊235┊        content: 'I made a group',
-┊346┊   ┊        createdAt: moment().subtract(2, 'weeks').unix(),
+┊   ┊236┊        createdAt: moment().subtract(2, 'weeks').toDate(),
 ┊347┊237┊        type: MessageType.TEXT,
+┊   ┊238┊        holders: [user1, user3, user4, user6],
 ┊348┊239┊        recipients: [
-┊349┊   ┊          {
-┊350┊   ┊            userId: 3,
-┊351┊   ┊            messageId: 1,
-┊352┊   ┊            chatId: 8,
-┊353┊   ┊            receivedAt: null,
-┊354┊   ┊            readAt: null,
-┊355┊   ┊          },
-┊356┊   ┊          {
-┊357┊   ┊            userId: 4,
-┊358┊   ┊            messageId: 1,
-┊359┊   ┊            chatId: 8,
-┊360┊   ┊            receivedAt: moment().subtract(2, 'weeks').add(1, 'minutes').unix(),
-┊361┊   ┊            readAt: moment().subtract(2, 'weeks').add(5, 'minutes').unix(),
-┊362┊   ┊          },
-┊363┊   ┊          {
-┊364┊   ┊            userId: 6,
-┊365┊   ┊            messageId: 1,
-┊366┊   ┊            chatId: 8,
-┊367┊   ┊            receivedAt: null,
-┊368┊   ┊            readAt: null,
-┊369┊   ┊          },
+┊   ┊240┊          new Recipient({
+┊   ┊241┊            user: user3,
+┊   ┊242┊          }),
+┊   ┊243┊          new Recipient({
+┊   ┊244┊            user: user4,
+┊   ┊245┊          }),
+┊   ┊246┊          new Recipient({
+┊   ┊247┊            user: user6,
+┊   ┊248┊          }),
 ┊370┊249┊        ],
-┊371┊   ┊        holderIds: [1, 3, 4, 6],
-┊372┊   ┊      },
-┊373┊   ┊      {
-┊374┊   ┊        id: 2,
-┊375┊   ┊        chatId: 8,
-┊376┊   ┊        senderId: 1,
-┊377┊   ┊        content: 'Ops, user 3 was not supposed to be here',
-┊378┊   ┊        createdAt: moment().subtract(2, 'weeks').add(2, 'minutes').unix(),
+┊   ┊250┊      }),
+┊   ┊251┊      new Message({
+┊   ┊252┊        sender: user1,
+┊   ┊253┊        content: 'Ops, Avery was not supposed to be here',
+┊   ┊254┊        createdAt: moment().subtract(2, 'weeks').add(2, 'minutes').toDate(),
 ┊379┊255┊        type: MessageType.TEXT,
+┊   ┊256┊        holders: [user1, user4, user6],
 ┊380┊257┊        recipients: [
-┊381┊   ┊          {
-┊382┊   ┊            userId: 4,
-┊383┊   ┊            messageId: 2,
-┊384┊   ┊            chatId: 8,
-┊385┊   ┊            receivedAt: moment().subtract(2, 'weeks').add(3, 'minutes').unix(),
-┊386┊   ┊            readAt: moment().subtract(2, 'weeks').add(5, 'minutes').unix(),
-┊387┊   ┊          },
-┊388┊   ┊          {
-┊389┊   ┊            userId: 6,
-┊390┊   ┊            messageId: 2,
-┊391┊   ┊            chatId: 8,
-┊392┊   ┊            receivedAt: null,
-┊393┊   ┊            readAt: null,
-┊394┊   ┊          },
+┊   ┊258┊          new Recipient({
+┊   ┊259┊            user: user4,
+┊   ┊260┊          }),
+┊   ┊261┊          new Recipient({
+┊   ┊262┊            user: user6,
+┊   ┊263┊          }),
 ┊395┊264┊        ],
-┊396┊   ┊        holderIds: [1, 4, 6],
-┊397┊   ┊      },
-┊398┊   ┊      {
-┊399┊   ┊        id: 3,
-┊400┊   ┊        chatId: 8,
-┊401┊   ┊        senderId: 4,
+┊   ┊265┊      }),
+┊   ┊266┊      new Message({
+┊   ┊267┊        sender: user4,
 ┊402┊268┊        content: 'Awesome!',
-┊403┊   ┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').unix(),
+┊   ┊269┊        createdAt: moment().subtract(2, 'weeks').add(10, 'minutes').toDate(),
 ┊404┊270┊        type: MessageType.TEXT,
+┊   ┊271┊        holders: [user1, user4, user6],
 ┊405┊272┊        recipients: [
-┊406┊   ┊          {
-┊407┊   ┊            userId: 1,
-┊408┊   ┊            messageId: 3,
-┊409┊   ┊            chatId: 8,
-┊410┊   ┊            receivedAt: null,
-┊411┊   ┊            readAt: null,
-┊412┊   ┊          },
-┊413┊   ┊          {
-┊414┊   ┊            userId: 6,
-┊415┊   ┊            messageId: 3,
-┊416┊   ┊            chatId: 8,
-┊417┊   ┊            receivedAt: null,
-┊418┊   ┊            readAt: null,
-┊419┊   ┊          },
+┊   ┊273┊          new Recipient({
+┊   ┊274┊            user: user1,
+┊   ┊275┊          }),
+┊   ┊276┊          new Recipient({
+┊   ┊277┊            user: user6,
+┊   ┊278┊          }),
 ┊420┊279┊        ],
-┊421┊   ┊        holderIds: [1, 4, 6],
-┊422┊   ┊      },
+┊   ┊280┊      }),
 ┊423┊281┊    ],
-┊424┊   ┊  },
-┊425┊   ┊  {
-┊426┊   ┊    id: 9,
-┊427┊   ┊    name: 'A user 5 group',
-┊428┊   ┊    picture: null,
-┊429┊   ┊    allTimeMemberIds: [6, 3],
-┊430┊   ┊    listingMemberIds: [6, 3],
-┊431┊   ┊    actualGroupMemberIds: [6, 3],
-┊432┊   ┊    adminIds: [6],
-┊433┊   ┊    ownerId: 6,
-┊434┊   ┊    messages: [],
-┊435┊   ┊  },
-┊436┊   ┊];
+┊   ┊282┊  }));
 ┊437┊283┊
-┊438┊   ┊export const db = {users, chats};
+┊   ┊284┊  await connection.manager.save(new Chat({
+┊   ┊285┊    name: 'Ray\'s group',
+┊   ┊286┊    allTimeMembers: [user3, user6],
+┊   ┊287┊    listingMembers: [user3, user6],
+┊   ┊288┊    actualGroupMembers: [user3, user6],
+┊   ┊289┊    admins: [user6],
+┊   ┊290┊    owner: user6,
+┊   ┊291┊  }));
+┊   ┊292┊}
```

[}]: #

It's time to deal with resolvers:

[{]: <helper> (diffStep "6.1" files="schema/resolvers.ts" module="server")

#### [Step 6.1: TypeORM with PostgreSQL](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/commit/10aa8a1)

##### Changed schema&#x2F;resolvers.ts
```diff
@@ -1,88 +1,122 @@
 ┊  1┊  1┊import { PubSub, withFilter } from 'apollo-server-express';
-┊  2┊   ┊import { ChatDb, db, MessageDb, MessageType, RecipientDb, UserDb } from "../db";
+┊   ┊  2┊import { MessageType } from "../db";
 ┊  3┊  3┊import { IResolvers, MessageAddedSubscriptionArgs } from "../types";
-┊  4┊   ┊import moment from "moment";
-┊  5┊   ┊
-┊  6┊   ┊let users = db.users;
-┊  7┊   ┊let chats = db.chats;
+┊   ┊  4┊import { User } from "../entity/User";
+┊   ┊  5┊import { Chat } from "../entity/Chat";
+┊   ┊  6┊import { Message } from "../entity/Message";
+┊   ┊  7┊import { Recipient } from "../entity/Recipient";
 ┊  8┊  8┊
 ┊  9┊  9┊export const pubsub = new PubSub();
 ┊ 10┊ 10┊
 ┊ 11┊ 11┊export const resolvers: IResolvers = {
 ┊ 12┊ 12┊  Query: {
 ┊ 13┊ 13┊    // Show all users for the moment.
-┊ 14┊   ┊    users: (obj, args, {user: currentUser}) => users.filter(user => user.id !== currentUser.id),
-┊ 15┊   ┊    chats: (obj, args, {user: currentUser}) => chats.filter(chat => chat.listingMemberIds.includes(currentUser.id)),
-┊ 16┊   ┊    chat: (obj, {chatId}) => chats.find(chat => chat.id === Number(chatId)) || null,
+┊   ┊ 14┊    users: async (obj, args, {user: currentUser, connection}) => {
+┊   ┊ 15┊      return await connection
+┊   ┊ 16┊        .createQueryBuilder(User, "user")
+┊   ┊ 17┊        .where('user.id != :id', {id: currentUser.id})
+┊   ┊ 18┊        .getMany();
+┊   ┊ 19┊    },
+┊   ┊ 20┊    chats: async (obj, args, {user: currentUser, connection}) => {
+┊   ┊ 21┊      return await connection
+┊   ┊ 22┊        .createQueryBuilder(Chat, "chat")
+┊   ┊ 23┊        .leftJoin('chat.listingMembers', 'listingMembers')
+┊   ┊ 24┊        .where('listingMembers.id = :id', {id: currentUser.id})
+┊   ┊ 25┊        .getMany();
+┊   ┊ 26┊    },
+┊   ┊ 27┊    chat: async (obj, {chatId}, {connection}) => {
+┊   ┊ 28┊      return await connection
+┊   ┊ 29┊        .createQueryBuilder(Chat, "chat")
+┊   ┊ 30┊        .whereInIds(chatId)
+┊   ┊ 31┊        .getOne();
+┊   ┊ 32┊    },
 ┊ 17┊ 33┊  },
 ┊ 18┊ 34┊  Mutation: {
-┊ 19┊   ┊    addChat: (obj, {recipientId}, {user: currentUser}) => {
-┊ 20┊   ┊      if (!users.find(user => user.id === Number(recipientId))) {
+┊   ┊ 35┊    addChat: async (obj, {recipientId}, {user: currentUser, connection}) => {
+┊   ┊ 36┊      const recipient = await connection
+┊   ┊ 37┊        .createQueryBuilder(User, "user")
+┊   ┊ 38┊        .whereInIds(recipientId)
+┊   ┊ 39┊        .getOne();
+┊   ┊ 40┊
+┊   ┊ 41┊      if (!recipient) {
 ┊ 21┊ 42┊        throw new Error(`Recipient ${recipientId} doesn't exist.`);
 ┊ 22┊ 43┊      }
 ┊ 23┊ 44┊
-┊ 24┊   ┊      const chat = chats.find(chat => !chat.name && chat.allTimeMemberIds.includes(currentUser.id) && chat.allTimeMemberIds.includes(Number(recipientId)));
+┊   ┊ 45┊      let chat = await connection
+┊   ┊ 46┊        .createQueryBuilder(Chat, "chat")
+┊   ┊ 47┊        .where('chat.name IS NULL')
+┊   ┊ 48┊        .innerJoin('chat.allTimeMembers', 'allTimeMembers1', 'allTimeMembers1.id = :currentUserId', {currentUserId: currentUser.id})
+┊   ┊ 49┊        .innerJoin('chat.allTimeMembers', 'allTimeMembers2', 'allTimeMembers2.id = :recipientId', {recipientId})
+┊   ┊ 50┊        .innerJoinAndSelect('chat.listingMembers', 'listingMembers')
+┊   ┊ 51┊        .getOne();
+┊   ┊ 52┊
 ┊ 25┊ 53┊      if (chat) {
-┊ 26┊   ┊        // Chat already exists. Both users are already in the allTimeMemberIds array
-┊ 27┊   ┊        const chatId = chat.id;
-┊ 28┊   ┊        if (!chat.listingMemberIds.includes(currentUser.id)) {
+┊   ┊ 54┊        // Chat already exists. Both users are already in the userIds array
+┊   ┊ 55┊        const listingMembers = await connection
+┊   ┊ 56┊          .createQueryBuilder(User, "user")
+┊   ┊ 57┊          .innerJoin('user.listingMemberChats', 'listingMemberChats', 'listingMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊ 58┊          .getMany();
+┊   ┊ 59┊
+┊   ┊ 60┊        if (!listingMembers.find(user => user.id === currentUser.id)) {
 ┊ 29┊ 61┊          // The chat isn't listed for the current user. Add him to the memberIds
-┊ 30┊   ┊          chat.listingMemberIds.push(currentUser.id);
-┊ 31┊   ┊          chats.find(chat => chat.id === chatId)!.listingMemberIds.push(currentUser.id);
-┊ 32┊   ┊          return chat;
+┊   ┊ 62┊          chat.listingMembers.push(currentUser);
+┊   ┊ 63┊          chat = await connection.getRepository(Chat).save(chat);
+┊   ┊ 64┊
+┊   ┊ 65┊          return chat || null;
 ┊ 33┊ 66┊        } else {
 ┊ 34┊ 67┊          throw new Error(`Chat already exists.`);
 ┊ 35┊ 68┊        }
 ┊ 36┊ 69┊      } else {
 ┊ 37┊ 70┊        // Create the chat
-┊ 38┊   ┊        const id = (chats.length && chats[chats.length - 1].id + 1) || 1;
-┊ 39┊   ┊        const chat: ChatDb = {
-┊ 40┊   ┊          id,
-┊ 41┊   ┊          name: null,
-┊ 42┊   ┊          picture: null,
-┊ 43┊   ┊          adminIds: null,
-┊ 44┊   ┊          ownerId: null,
-┊ 45┊   ┊          allTimeMemberIds: [currentUser.id, Number(recipientId)],
+┊   ┊ 71┊        chat = await connection.getRepository(Chat).save(new Chat({
+┊   ┊ 72┊          allTimeMembers: [currentUser, recipient],
 ┊ 46┊ 73┊          // Chat will not be listed to the other user until the first message gets written
-┊ 47┊   ┊          listingMemberIds: [currentUser.id],
-┊ 48┊   ┊          actualGroupMemberIds: null,
-┊ 49┊   ┊          messages: [],
-┊ 50┊   ┊        };
-┊ 51┊   ┊        chats.push(chat);
+┊   ┊ 74┊          listingMembers: [currentUser],
+┊   ┊ 75┊        }));
 ┊ 52┊ 76┊
-┊ 53┊   ┊        return chat;
+┊   ┊ 77┊        return chat || null;
 ┊ 54┊ 78┊      }
 ┊ 55┊ 79┊    },
-┊ 56┊   ┊    addGroup: (obj, {recipientIds, groupName}, {user: currentUser}) => {
-┊ 57┊   ┊      recipientIds.forEach(recipientId => {
-┊ 58┊   ┊        if (!users.find(user => user.id === Number(recipientId))) {
+┊   ┊ 80┊    addGroup: async (obj, {recipientIds, groupName}, {user: currentUser, connection}) => {
+┊   ┊ 81┊      let recipients: User[] = [];
+┊   ┊ 82┊      for (let recipientId of recipientIds) {
+┊   ┊ 83┊        const recipient = await connection
+┊   ┊ 84┊          .createQueryBuilder(User, "user")
+┊   ┊ 85┊          .whereInIds(recipientId)
+┊   ┊ 86┊          .getOne();
+┊   ┊ 87┊        if (!recipient) {
 ┊ 59┊ 88┊          throw new Error(`Recipient ${recipientId} doesn't exist.`);
 ┊ 60┊ 89┊        }
-┊ 61┊   ┊      });
+┊   ┊ 90┊        recipients.push(recipient);
+┊   ┊ 91┊      }
 ┊ 62┊ 92┊
-┊ 63┊   ┊      const id = (chats.length && chats[chats.length - 1].id + 1) || 1;
-┊ 64┊   ┊      const chat: ChatDb = {
-┊ 65┊   ┊        id,
+┊   ┊ 93┊      const chat = await connection.getRepository(Chat).save(new Chat({
 ┊ 66┊ 94┊        name: groupName,
-┊ 67┊   ┊        picture: null,
-┊ 68┊   ┊        adminIds: [currentUser.id],
-┊ 69┊   ┊        ownerId: currentUser.id,
-┊ 70┊   ┊        allTimeMemberIds: [currentUser.id, ...recipientIds.map(id => Number(id))],
-┊ 71┊   ┊        listingMemberIds: [currentUser.id, ...recipientIds.map(id => Number(id))],
-┊ 72┊   ┊        actualGroupMemberIds: [currentUser.id, ...recipientIds.map(id => Number(id))],
-┊ 73┊   ┊        messages: [],
-┊ 74┊   ┊      };
-┊ 75┊   ┊      chats.push(chat);
+┊   ┊ 95┊        admins: [currentUser],
+┊   ┊ 96┊        owner: currentUser,
+┊   ┊ 97┊        allTimeMembers: [...recipients, currentUser],
+┊   ┊ 98┊        listingMembers: [...recipients, currentUser],
+┊   ┊ 99┊        actualGroupMembers: [...recipients, currentUser],
+┊   ┊100┊      }));
 ┊ 76┊101┊
 ┊ 77┊102┊      pubsub.publish('chatAdded', {
 ┊ 78┊103┊        creatorId: currentUser.id,
 ┊ 79┊104┊        chatAdded: chat,
 ┊ 80┊105┊      });
 ┊ 81┊106┊
-┊ 82┊   ┊      return chat;
+┊   ┊107┊      return chat || null;
 ┊ 83┊108┊    },
-┊ 84┊   ┊    removeChat: (obj, {chatId}, {user: currentUser}) => {
-┊ 85┊   ┊      const chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊109┊    removeChat: async (obj, {chatId}, {user: currentUser, connection}) => {
+┊   ┊110┊      const chat = await connection
+┊   ┊111┊        .createQueryBuilder(Chat, "chat")
+┊   ┊112┊        .whereInIds(Number(chatId))
+┊   ┊113┊        .innerJoinAndSelect('chat.listingMembers', 'listingMembers')
+┊   ┊114┊        .leftJoinAndSelect('chat.actualGroupMembers', 'actualGroupMembers')
+┊   ┊115┊        .leftJoinAndSelect('chat.admins', 'admins')
+┊   ┊116┊        .leftJoinAndSelect('chat.owner', 'owner')
+┊   ┊117┊        .leftJoinAndSelect('chat.messages', 'messages')
+┊   ┊118┊        .leftJoinAndSelect('messages.holders', 'holders')
+┊   ┊119┊        .getOne();
 ┊ 86┊120┊
 ┊ 87┊121┊      if (!chat) {
 ┊ 88┊122┊        throw new Error(`The chat ${chatId} doesn't exist.`);
```
```diff
@@ -90,186 +124,188 @@
 ┊ 90┊124┊
 ┊ 91┊125┊      if (!chat.name) {
 ┊ 92┊126┊        // Chat
-┊ 93┊   ┊        if (!chat.listingMemberIds.includes(currentUser.id)) {
-┊ 94┊   ┊          throw new Error(`The user is not a member of the chat ${chatId}.`);
+┊   ┊127┊        if (!chat.listingMembers.find(user => user.id === currentUser.id)) {
+┊   ┊128┊          throw new Error(`The user is not a listing member of the chat ${chatId}.`);
 ┊ 95┊129┊        }
 ┊ 96┊130┊
 ┊ 97┊131┊        // Instead of chaining map and filter we can loop once using reduce
-┊ 98┊   ┊        const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
-┊ 99┊   ┊          // Remove the current user from the message holders
-┊100┊   ┊          message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
+┊   ┊132┊        chat.messages = await chat.messages.reduce<Promise<Message[]>>(async (filtered$, message) => {
+┊   ┊133┊          const filtered = await filtered$;
 ┊101┊134┊
-┊102┊   ┊          if (message.holderIds.length !== 0) {
+┊   ┊135┊          message.holders = message.holders.filter(user => user.id !== currentUser.id);
+┊   ┊136┊
+┊   ┊137┊          if (message.holders.length !== 0) {
+┊   ┊138┊            // Remove the current user from the message holders
+┊   ┊139┊            await connection.getRepository(Message).save(message);
 ┊103┊140┊            filtered.push(message);
-┊104┊   ┊          } // else discard the message
+┊   ┊141┊          } else {
+┊   ┊142┊            // Simply remove the message
+┊   ┊143┊            const recipients = await connection
+┊   ┊144┊              .createQueryBuilder(Recipient, "recipient")
+┊   ┊145┊              .innerJoinAndSelect('recipient.message', 'message', 'message.id = :messageId', {messageId: message.id})
+┊   ┊146┊              .innerJoinAndSelect('recipient.user', 'user')
+┊   ┊147┊              .getMany();
+┊   ┊148┊            for (let recipient of recipients) {
+┊   ┊149┊              await connection.getRepository(Recipient).remove(recipient);
+┊   ┊150┊            }
+┊   ┊151┊            await connection.getRepository(Message).remove(message);
+┊   ┊152┊          }
 ┊105┊153┊
 ┊106┊154┊          return filtered;
-┊107┊   ┊        }, []);
+┊   ┊155┊        }, Promise.resolve([]));
 ┊108┊156┊
 ┊109┊157┊        // Remove the current user from who gets the chat listed. The chat will no longer appear in his list
-┊110┊   ┊        const listingMemberIds = chat.listingMemberIds.filter(listingId => listingId !== currentUser.id);
+┊   ┊158┊        chat.listingMembers = chat.listingMembers.filter(user => user.id !== currentUser.id);
 ┊111┊159┊
 ┊112┊160┊        // Check how many members are left
-┊113┊   ┊        if (listingMemberIds.length === 0) {
+┊   ┊161┊        if (chat.listingMembers.length === 0) {
 ┊114┊162┊          // Delete the chat
-┊115┊   ┊          chats = chats.filter(chat => chat.id !== Number(chatId));
+┊   ┊163┊          await connection.getRepository(Chat).remove(chat);
 ┊116┊164┊        } else {
 ┊117┊165┊          // Update the chat
-┊118┊   ┊          chats = chats.map(chat => {
-┊119┊   ┊            if (chat.id === Number(chatId)) {
-┊120┊   ┊              chat = {...chat, listingMemberIds, messages};
-┊121┊   ┊            }
-┊122┊   ┊            return chat;
-┊123┊   ┊          });
+┊   ┊166┊          await connection.getRepository(Chat).save(chat);
 ┊124┊167┊        }
 ┊125┊168┊        return chatId;
 ┊126┊169┊      } else {
 ┊127┊170┊        // Group
-┊128┊   ┊        if (chat.ownerId !== currentUser.id) {
-┊129┊   ┊          throw new Error(`Group ${chatId} is not owned by the user.`);
-┊130┊   ┊        }
 ┊131┊171┊
 ┊132┊172┊        // Instead of chaining map and filter we can loop once using reduce
-┊133┊   ┊        const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
-┊134┊   ┊          // Remove the current user from the message holders
-┊135┊   ┊          message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
+┊   ┊173┊        chat.messages = await chat.messages.reduce<Promise<Message[]>>(async (filtered$, message) => {
+┊   ┊174┊          const filtered = await filtered$;
+┊   ┊175┊
+┊   ┊176┊          message.holders = message.holders.filter(user => user.id !== currentUser.id);
 ┊136┊177┊
-┊137┊   ┊          if (message.holderIds.length !== 0) {
+┊   ┊178┊          if (message.holders.length !== 0) {
+┊   ┊179┊            // Remove the current user from the message holders
+┊   ┊180┊            await connection.getRepository(Message).save(message);
 ┊138┊181┊            filtered.push(message);
-┊139┊   ┊          } // else discard the message
+┊   ┊182┊          } else {
+┊   ┊183┊            // Simply remove the message
+┊   ┊184┊            const recipients = await connection
+┊   ┊185┊              .createQueryBuilder(Recipient, "recipient")
+┊   ┊186┊              .innerJoinAndSelect('recipient.message', 'message', 'message.id = :messageId', {messageId: message.id})
+┊   ┊187┊              .innerJoinAndSelect('recipient.user', 'user')
+┊   ┊188┊              .getMany();
+┊   ┊189┊            for (let recipient of recipients) {
+┊   ┊190┊              await connection.getRepository(Recipient).remove(recipient);
+┊   ┊191┊            }
+┊   ┊192┊            await connection.getRepository(Message).remove(message);
+┊   ┊193┊          }
 ┊140┊194┊
 ┊141┊195┊          return filtered;
-┊142┊   ┊        }, []);
+┊   ┊196┊        }, Promise.resolve([]));
 ┊143┊197┊
 ┊144┊198┊        // Remove the current user from who gets the group listed. The group will no longer appear in his list
-┊145┊   ┊        const listingMemberIds = chat.listingMemberIds.filter(listingId => listingId !== currentUser.id);
+┊   ┊199┊        chat.listingMembers = chat.listingMembers.filter(user => user.id !== currentUser.id);
 ┊146┊200┊
 ┊147┊201┊        // Check how many members (including previous ones who can still access old messages) are left
-┊148┊   ┊        if (listingMemberIds.length === 0) {
+┊   ┊202┊        if (chat.listingMembers.length === 0) {
 ┊149┊203┊          // Remove the group
-┊150┊   ┊          chats = chats.filter(chat => chat.id !== Number(chatId));
+┊   ┊204┊          await connection.getRepository(Chat).remove(chat);
 ┊151┊205┊        } else {
 ┊152┊206┊          // Update the group
 ┊153┊207┊
 ┊154┊208┊          // Remove the current user from the chat members. He is no longer a member of the group
-┊155┊   ┊          const actualGroupMemberIds = chat.actualGroupMemberIds!.filter(memberId => memberId !== currentUser.id);
+┊   ┊209┊          chat.actualGroupMembers = chat.actualGroupMembers && chat.actualGroupMembers.filter(user => user.id !== currentUser.id);
 ┊156┊210┊          // Remove the current user from the chat admins
-┊157┊   ┊          const adminIds = chat.adminIds!.filter(memberId => memberId !== currentUser.id);
-┊158┊   ┊          // Set the owner id to be null. A null owner means the group is read-only
-┊159┊   ┊          let ownerId: number | null = null;
-┊160┊   ┊
-┊161┊   ┊          // Check if there is any admin left
-┊162┊   ┊          if (adminIds!.length) {
-┊163┊   ┊            // Pick an admin as the new owner. The group is no longer read-only
-┊164┊   ┊            ownerId = chat.adminIds![0];
-┊165┊   ┊          }
+┊   ┊211┊          chat.admins = chat.admins && chat.admins.filter(user => user.id !== currentUser.id);
+┊   ┊212┊          // If there are no more admins left the group goes read only
+┊   ┊213┊          chat.owner = chat.admins && chat.admins[0] || null; // A null owner means the group is read-only
 ┊166┊214┊
-┊167┊   ┊          chats = chats.map(chat => {
-┊168┊   ┊            if (chat.id === Number(chatId)) {
-┊169┊   ┊              chat = {...chat, messages, listingMemberIds, actualGroupMemberIds, adminIds, ownerId};
-┊170┊   ┊            }
-┊171┊   ┊            return chat;
-┊172┊   ┊          });
+┊   ┊215┊          await connection.getRepository(Chat).save(chat);
 ┊173┊216┊        }
 ┊174┊217┊        return chatId;
 ┊175┊218┊      }
 ┊176┊219┊    },
-┊177┊   ┊    addMessage: (obj, {chatId, content}, {user: currentUser}) => {
+┊   ┊220┊    addMessage: async (obj, {chatId, content}, {user: currentUser, connection}) => {
 ┊178┊221┊      if (content === null || content === '') {
 ┊179┊222┊        throw new Error(`Cannot add empty or null messages.`);
 ┊180┊223┊      }
 ┊181┊224┊
-┊182┊   ┊      let chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊225┊      let chat = await connection
+┊   ┊226┊        .createQueryBuilder(Chat, "chat")
+┊   ┊227┊        .whereInIds(chatId)
+┊   ┊228┊        .innerJoinAndSelect('chat.allTimeMembers', 'allTimeMembers')
+┊   ┊229┊        .innerJoinAndSelect('chat.listingMembers', 'listingMembers')
+┊   ┊230┊        .leftJoinAndSelect('chat.actualGroupMembers', 'actualGroupMembers')
+┊   ┊231┊        .getOne();
 ┊183┊232┊
 ┊184┊233┊      if (!chat) {
 ┊185┊234┊        throw new Error(`Cannot find chat ${chatId}.`);
 ┊186┊235┊      }
 ┊187┊236┊
-┊188┊   ┊      let holderIds = chat.listingMemberIds;
+┊   ┊237┊      let holders: User[];
 ┊189┊238┊
 ┊190┊239┊      if (!chat.name) {
 ┊191┊240┊        // Chat
-┊192┊   ┊        if (!chat.listingMemberIds.find(listingId => listingId === currentUser.id)) {
+┊   ┊241┊        if (!chat.listingMembers.map(user => user.id).includes(currentUser.id)) {
 ┊193┊242┊          throw new Error(`The chat ${chatId} must be listed for the current user before adding a message.`);
 ┊194┊243┊        }
 ┊195┊244┊
-┊196┊   ┊        const recipientId = chat.allTimeMemberIds.filter(userId => userId !== currentUser.id)[0];
+┊   ┊245┊        const recipientUser = chat.allTimeMembers.find(user => user.id !== currentUser.id);
 ┊197┊246┊
-┊198┊   ┊        if (!chat.listingMemberIds.find(listingId => listingId === recipientId)) {
-┊199┊   ┊          // Chat is not listed for the recipient. Add him to the listingMemberIds
-┊200┊   ┊          const listingMemberIds = chat.listingMemberIds.concat(recipientId);
+┊   ┊247┊        if (!recipientUser) {
+┊   ┊248┊          throw new Error(`Cannot find recipient user.`);
+┊   ┊249┊        }
 ┊201┊250┊
-┊202┊   ┊          chats = chats.map(chat => {
-┊203┊   ┊            if (chat.id === Number(chatId)) {
-┊204┊   ┊              chat = {...chat, listingMemberIds};
-┊205┊   ┊            }
-┊206┊   ┊            return chat;
-┊207┊   ┊          });
+┊   ┊251┊        if (!chat.listingMembers.find(user => user.id === recipientUser.id)) {
+┊   ┊252┊          // Chat is not listed for the recipient. Add him to the listingIds
+┊   ┊253┊          chat.listingMembers.push(recipientUser);
 ┊208┊254┊
-┊209┊   ┊          holderIds = listingMemberIds;
+┊   ┊255┊          await connection.getRepository(Chat).save(chat);
 ┊210┊256┊
 ┊211┊257┊          pubsub.publish('chatAdded', {
 ┊212┊258┊            creatorId: currentUser.id,
 ┊213┊259┊            chatAdded: chat,
 ┊214┊260┊          });
 ┊215┊261┊        }
+┊   ┊262┊
+┊   ┊263┊        holders = chat.listingMembers;
 ┊216┊264┊      } else {
 ┊217┊265┊        // Group
-┊218┊   ┊        if (!chat.actualGroupMemberIds!.find(memberId => memberId === currentUser.id)) {
+┊   ┊266┊        if (!chat.actualGroupMembers || !chat.actualGroupMembers.find(user => user.id === currentUser.id)) {
 ┊219┊267┊          throw new Error(`The user is not a member of the group ${chatId}. Cannot add message.`);
 ┊220┊268┊        }
 ┊221┊269┊
-┊222┊   ┊        holderIds = chat.actualGroupMemberIds!;
+┊   ┊270┊        holders = chat.actualGroupMembers;
 ┊223┊271┊      }
 ┊224┊272┊
-┊225┊   ┊      const id = (chat.messages.length && chat.messages[chat.messages.length - 1].id + 1) || 1;
-┊226┊   ┊
-┊227┊   ┊      let recipients: RecipientDb[] = [];
-┊228┊   ┊
-┊229┊   ┊      holderIds.forEach(holderId => {
-┊230┊   ┊        if (holderId !== currentUser.id) {
-┊231┊   ┊          recipients.push({
-┊232┊   ┊            userId: holderId,
-┊233┊   ┊            messageId: id,
-┊234┊   ┊            chatId: Number(chatId),
-┊235┊   ┊            receivedAt: null,
-┊236┊   ┊            readAt: null,
-┊237┊   ┊          });
-┊238┊   ┊        }
-┊239┊   ┊      });
-┊240┊   ┊
-┊241┊   ┊      const message: MessageDb = {
-┊242┊   ┊        id,
-┊243┊   ┊        chatId: Number(chatId),
-┊244┊   ┊        senderId: currentUser.id,
+┊   ┊273┊      const message = await connection.getRepository(Message).save(new Message({
+┊   ┊274┊        chat,
+┊   ┊275┊        sender: currentUser,
 ┊245┊276┊        content,
-┊246┊   ┊        createdAt: moment().unix(),
 ┊247┊277┊        type: MessageType.TEXT,
-┊248┊   ┊        recipients,
-┊249┊   ┊        holderIds,
-┊250┊   ┊      };
-┊251┊   ┊
-┊252┊   ┊      chats = chats.map(chat => {
-┊253┊   ┊        if (chat.id === Number(chatId)) {
-┊254┊   ┊          chat = {...chat, messages: chat.messages.concat(message)}
-┊255┊   ┊        }
-┊256┊   ┊        return chat;
-┊257┊   ┊      });
+┊   ┊278┊        holders,
+┊   ┊279┊        recipients: holders.reduce<Recipient[]>((filtered, user) => {
+┊   ┊280┊          if (user.id !== currentUser.id) {
+┊   ┊281┊            filtered.push(new Recipient({
+┊   ┊282┊              user,
+┊   ┊283┊            }));
+┊   ┊284┊          }
+┊   ┊285┊          return filtered;
+┊   ┊286┊        }, []),
+┊   ┊287┊      }));
 ┊258┊288┊
 ┊259┊289┊      pubsub.publish('messageAdded', {
 ┊260┊290┊        messageAdded: message,
 ┊261┊291┊      });
 ┊262┊292┊
-┊263┊   ┊      return message;
+┊   ┊293┊      return message || null;
 ┊264┊294┊    },
-┊265┊   ┊    removeMessages: (obj, {chatId, messageIds, all}, {user: currentUser}) => {
-┊266┊   ┊      const chat = chats.find(chat => chat.id === Number(chatId));
+┊   ┊295┊    removeMessages: async (obj, {chatId, messageIds, all}, {user: currentUser, connection}) => {
+┊   ┊296┊      const chat = await connection
+┊   ┊297┊        .createQueryBuilder(Chat, "chat")
+┊   ┊298┊        .whereInIds(chatId)
+┊   ┊299┊        .innerJoinAndSelect('chat.listingMembers', 'listingMembers')
+┊   ┊300┊        .innerJoinAndSelect('chat.messages', 'messages')
+┊   ┊301┊        .innerJoinAndSelect('messages.holders', 'holders')
+┊   ┊302┊        .getOne();
 ┊267┊303┊
 ┊268┊304┊      if (!chat) {
 ┊269┊305┊        throw new Error(`Cannot find chat ${chatId}.`);
 ┊270┊306┊      }
 ┊271┊307┊
-┊272┊   ┊      if (!chat.listingMemberIds.find(listingId => listingId === currentUser.id)) {
+┊   ┊308┊      if (!chat.listingMembers.find(user => user.id === currentUser.id)) {
 ┊273┊309┊        throw new Error(`The chat/group ${chatId} is not listed for the current user, so there is nothing to delete.`);
 ┊274┊310┊      }
 ┊275┊311┊
```
```diff
@@ -277,79 +313,166 @@
 ┊277┊313┊        throw new Error(`Cannot specify both 'all' and 'messageIds'.`);
 ┊278┊314┊      }
 ┊279┊315┊
+┊   ┊316┊      if (!all && !(messageIds && messageIds.length)) {
+┊   ┊317┊        throw new Error(`'all' and 'messageIds' cannot be both null`);
+┊   ┊318┊      }
+┊   ┊319┊
 ┊280┊320┊      let deletedIds: string[] = [];
-┊281┊   ┊      chats = chats.map(chat => {
-┊282┊   ┊        if (chat.id === Number(chatId)) {
-┊283┊   ┊          // Instead of chaining map and filter we can loop once using reduce
-┊284┊   ┊          const messages = chat.messages.reduce<MessageDb[]>((filtered, message) => {
-┊285┊   ┊            if (all || messageIds!.map(Number).includes(message.id)) {
-┊286┊   ┊              deletedIds.push(String(message.id));
-┊287┊   ┊              // Remove the current user from the message holders
-┊288┊   ┊              message.holderIds = message.holderIds.filter(holderId => holderId !== currentUser.id);
-┊289┊   ┊            }
+┊   ┊321┊      // Instead of chaining map and filter we can loop once using reduce
+┊   ┊322┊      chat.messages = await chat.messages.reduce<Promise<Message[]>>(async (filtered$, message) => {
+┊   ┊323┊        const filtered = await filtered$;
 ┊290┊324┊
-┊291┊   ┊            if (message.holderIds.length !== 0) {
-┊292┊   ┊              filtered.push(message);
-┊293┊   ┊            } // else discard the message
+┊   ┊325┊        if (all || messageIds!.map(Number).includes(message.id)) {
+┊   ┊326┊          deletedIds.push(String(message.id));
+┊   ┊327┊          // Remove the current user from the message holders
+┊   ┊328┊          message.holders = message.holders.filter(user => user.id !== currentUser.id);
 ┊294┊329┊
-┊295┊   ┊            return filtered;
-┊296┊   ┊          }, []);
-┊297┊   ┊          chat = {...chat, messages};
 ┊298┊330┊        }
-┊299┊   ┊        return chat;
-┊300┊   ┊      });
+┊   ┊331┊
+┊   ┊332┊        if (message.holders.length !== 0) {
+┊   ┊333┊          // Remove the current user from the message holders
+┊   ┊334┊          await connection.getRepository(Message).save(message);
+┊   ┊335┊          filtered.push(message);
+┊   ┊336┊        } else {
+┊   ┊337┊          // Simply remove the message
+┊   ┊338┊          const recipients = await connection
+┊   ┊339┊            .createQueryBuilder(Recipient, "recipient")
+┊   ┊340┊            .innerJoinAndSelect('recipient.message', 'message', 'message.id = :messageId', {messageId: message.id})
+┊   ┊341┊            .innerJoinAndSelect('recipient.user', 'user')
+┊   ┊342┊            .getMany();
+┊   ┊343┊          for (let recipient of recipients) {
+┊   ┊344┊            await connection.getRepository(Recipient).remove(recipient);
+┊   ┊345┊          }
+┊   ┊346┊          await connection.getRepository(Message).remove(message);
+┊   ┊347┊        }
+┊   ┊348┊
+┊   ┊349┊        return filtered;
+┊   ┊350┊      }, Promise.resolve([]));
+┊   ┊351┊
+┊   ┊352┊      await connection.getRepository(Chat).save(chat);
+┊   ┊353┊
 ┊301┊354┊      return deletedIds;
 ┊302┊355┊    },
 ┊303┊356┊  },
 ┊304┊357┊  Subscription: {
 ┊305┊358┊    messageAdded: {
 ┊306┊359┊      subscribe: withFilter(() => pubsub.asyncIterator('messageAdded'),
-┊307┊   ┊        ({messageAdded}: {messageAdded: MessageDb & {chat: {id: number}}}, {chatId}: MessageAddedSubscriptionArgs, {user: currentUser}: { user: UserDb }) => {
+┊   ┊360┊        ({messageAdded}: {messageAdded: Message}, {chatId}: MessageAddedSubscriptionArgs, {user: currentUser}: { user: User }) => {
 ┊308┊361┊          return (!chatId || messageAdded.chat.id === Number(chatId)) &&
-┊309┊   ┊            !!messageAdded.recipients.find((recipient: RecipientDb) => recipient.userId === currentUser.id);
+┊   ┊362┊            !!messageAdded.recipients.find((recipient: Recipient) => recipient.user.id === currentUser.id);
 ┊310┊363┊        }),
 ┊311┊364┊    },
 ┊312┊365┊    chatAdded: {
 ┊313┊366┊      subscribe: withFilter(() => pubsub.asyncIterator('chatAdded'),
-┊314┊   ┊        ({creatorId, chatAdded}: {creatorId: string, chatAdded: ChatDb}, variables: any, {user: currentUser}: { user: UserDb }) => {
-┊315┊   ┊          return Number(creatorId) !== currentUser.id && !chatAdded.listingMemberIds.includes(currentUser.id);
+┊   ┊367┊        ({creatorId, chatAdded}: {creatorId: string, chatAdded: Chat}, variables, {user: currentUser}: { user: User }) => {
+┊   ┊368┊          return Number(creatorId) !== currentUser.id &&
+┊   ┊369┊            !!chatAdded.listingMembers.find((user: User) => user.id === currentUser.id);
 ┊316┊370┊        }),
 ┊317┊371┊    }
 ┊318┊372┊  },
 ┊319┊373┊  Chat: {
-┊320┊   ┊    name: (chat, args, {user: currentUser}) => chat.name ? chat.name : users
-┊321┊   ┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.name,
-┊322┊   ┊    picture: (chat, args, {user: currentUser}) => chat.name ? chat.picture : users
-┊323┊   ┊      .find(user => user.id === chat.allTimeMemberIds.find(userId => userId !== currentUser.id))!.picture,
-┊324┊   ┊    allTimeMembers: (chat) => users.filter(user => chat.allTimeMemberIds.includes(user.id)),
-┊325┊   ┊    listingMembers: (chat) => users.filter(user => chat.listingMemberIds.includes(user.id)),
-┊326┊   ┊    actualGroupMembers: (chat) => users.filter(user => chat.actualGroupMemberIds && chat.actualGroupMemberIds.includes(user.id)),
-┊327┊   ┊    admins: (chat) => users.filter(user => chat.adminIds && chat.adminIds.includes(user.id)),
-┊328┊   ┊    owner: (chat) => users.find(user => chat.ownerId === user.id) || null,
-┊329┊   ┊    messages: (chat, {amount = 0}, {user: currentUser}) => {
-┊330┊   ┊      const messages = chat.messages
-┊331┊   ┊      .filter(message => message.holderIds.includes(currentUser.id))
-┊332┊   ┊      .sort((a, b) => b.createdAt - a.createdAt) || [];
-┊333┊   ┊      return (amount ? messages.slice(0, amount) : messages).reverse();
+┊   ┊374┊    name: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊375┊      if (chat.name) {
+┊   ┊376┊        return chat.name;
+┊   ┊377┊      }
+┊   ┊378┊      const user = await connection
+┊   ┊379┊        .createQueryBuilder(User, "user")
+┊   ┊380┊        .where('user.id != :userId', {userId: currentUser.id})
+┊   ┊381┊        .innerJoin('user.allTimeMemberChats', 'allTimeMemberChats', 'allTimeMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊382┊        .getOne();
+┊   ┊383┊      return user && user.name || null;
+┊   ┊384┊    },
+┊   ┊385┊    picture: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊386┊      if (chat.name) {
+┊   ┊387┊        return chat.picture;
+┊   ┊388┊      }
+┊   ┊389┊      const user = await connection
+┊   ┊390┊        .createQueryBuilder(User, "user")
+┊   ┊391┊        .where('user.id != :userId', {userId: currentUser.id})
+┊   ┊392┊        .innerJoin('user.allTimeMemberChats', 'allTimeMemberChats', 'allTimeMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊393┊        .getOne();
+┊   ┊394┊      return user ? user.picture : null;
+┊   ┊395┊    },
+┊   ┊396┊    allTimeMembers: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊397┊      return await connection
+┊   ┊398┊        .createQueryBuilder(User, "user")
+┊   ┊399┊        .innerJoin('user.allTimeMemberChats', 'allTimeMemberChats', 'allTimeMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊400┊        .getMany();
+┊   ┊401┊    },
+┊   ┊402┊    listingMembers: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊403┊      return await connection
+┊   ┊404┊        .createQueryBuilder(User, "user")
+┊   ┊405┊        .innerJoin('user.listingMemberChats', 'listingMemberChats', 'listingMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊406┊        .getMany();
+┊   ┊407┊    },
+┊   ┊408┊    actualGroupMembers: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊409┊      return await connection
+┊   ┊410┊        .createQueryBuilder(User, "user")
+┊   ┊411┊        .innerJoin('user.actualGroupMemberChats', 'actualGroupMemberChats', 'actualGroupMemberChats.id = :chatId', {chatId: chat.id})
+┊   ┊412┊        .getMany();
+┊   ┊413┊    },
+┊   ┊414┊    admins: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊415┊      return await connection
+┊   ┊416┊        .createQueryBuilder(User, "user")
+┊   ┊417┊        .innerJoin('user.adminChats', 'adminChats', 'adminChats.id = :chatId', {chatId: chat.id})
+┊   ┊418┊        .getMany();
+┊   ┊419┊    },
+┊   ┊420┊    owner: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊421┊      return await connection
+┊   ┊422┊        .createQueryBuilder(User, "user")
+┊   ┊423┊        .innerJoin('user.ownerChats', 'ownerChats', 'ownerChats.id = :chatId', {chatId: chat.id})
+┊   ┊424┊        .getOne() || null;
+┊   ┊425┊    },
+┊   ┊426┊    messages: async (chat, {amount = 0}: {amount: number}, {user: currentUser, connection}) => {
+┊   ┊427┊      const query = connection
+┊   ┊428┊        .createQueryBuilder(Message, "message")
+┊   ┊429┊        .innerJoin('message.chat', 'chat', 'chat.id = :chatId', {chatId: chat.id})
+┊   ┊430┊        .innerJoin('message.holders', 'holders', 'holders.id = :userId', {userId: currentUser.id})
+┊   ┊431┊        .orderBy({"message.createdAt": "DESC"});
+┊   ┊432┊      return (amount ? await query.take(amount).getMany() : await query.getMany()).reverse();
+┊   ┊433┊    },
+┊   ┊434┊    unreadMessages: async (chat, args, {user: currentUser, connection}) => {
+┊   ┊435┊      return await connection
+┊   ┊436┊        .createQueryBuilder(Message, "message")
+┊   ┊437┊        .innerJoin('message.chat', 'chat', 'chat.id = :chatId', {chatId: chat.id})
+┊   ┊438┊        .innerJoin('message.recipients', 'recipients', 'recipients.user.id = :userId AND recipients.readAt IS NULL', {userId: currentUser.id})
+┊   ┊439┊        .getCount();
 ┊334┊440┊    },
-┊335┊   ┊    unreadMessages: (chat, args, {user: currentUser}) => chat.messages
-┊336┊   ┊      .filter(message => message.holderIds.includes(currentUser.id) &&
-┊337┊   ┊        message.recipients.find(recipient => recipient.userId === currentUser.id && !recipient.readAt))
-┊338┊   ┊      .length,
 ┊339┊441┊    isGroup: (chat) => !!chat.name,
 ┊340┊442┊  },
 ┊341┊443┊  Message: {
-┊342┊   ┊    chat: (message) => chats.find(chat => message.chatId === chat.id)!,
-┊343┊   ┊    sender: (message) => users.find(user => user.id === message.senderId)!,
-┊344┊   ┊    holders: (message) => users.filter(user => message.holderIds.includes(user.id)),
-┊345┊   ┊    ownership: (message, args, {user: currentUser}) => message.senderId === currentUser.id,
-┊346┊   ┊  },
-┊347┊   ┊  Recipient: {
-┊348┊   ┊    user: (recipient) => users.find(user => recipient.userId === user.id)!,
-┊349┊   ┊    message: (recipient) => {
-┊350┊   ┊      const chat = chats.find(chat => recipient.chatId === chat.id)!;
-┊351┊   ┊      return chat.messages.find(message => recipient.messageId === message.id)!;
+┊   ┊444┊    sender: async (message: Message, args, {user: currentUser, connection}) => {
+┊   ┊445┊      return (await connection
+┊   ┊446┊        .createQueryBuilder(User, "user")
+┊   ┊447┊        .innerJoin('user.senderMessages', 'senderMessages', 'senderMessages.id = :messageId', {messageId: message.id})
+┊   ┊448┊        .getOne())!;
+┊   ┊449┊    },
+┊   ┊450┊    ownership: async (message: Message, args, {user: currentUser, connection}) => {
+┊   ┊451┊      return !!(await connection
+┊   ┊452┊        .createQueryBuilder(User, "user")
+┊   ┊453┊        .whereInIds(currentUser.id)
+┊   ┊454┊        .innerJoin('user.senderMessages', 'senderMessages', 'senderMessages.id = :messageId', {messageId: message.id})
+┊   ┊455┊        .getCount());
+┊   ┊456┊    },
+┊   ┊457┊    recipients: async (message: Message, args, {user: currentUser, connection}) => {
+┊   ┊458┊      return await connection
+┊   ┊459┊        .createQueryBuilder(Recipient, "recipient")
+┊   ┊460┊        .innerJoinAndSelect('recipient.message', 'message', 'message.id = :messageId', {messageId: message.id})
+┊   ┊461┊        .innerJoinAndSelect('recipient.user', 'user')
+┊   ┊462┊        .innerJoinAndSelect('recipient.chat', 'chat')
+┊   ┊463┊        .getMany();
+┊   ┊464┊    },
+┊   ┊465┊    holders: async (message: Message, args, {user: currentUser, connection}) => {
+┊   ┊466┊      return await connection
+┊   ┊467┊        .createQueryBuilder(User, "user")
+┊   ┊468┊        .innerJoin('user.holderMessages', 'holderMessages', 'holderMessages.id = :messageId', {messageId: message.id})
+┊   ┊469┊        .getMany();
+┊   ┊470┊    },
+┊   ┊471┊    chat: async (message: Message, args, {user: currentUser, connection})=> {
+┊   ┊472┊      return (await connection
+┊   ┊473┊        .createQueryBuilder(Chat, "chat")
+┊   ┊474┊        .innerJoin('chat.messages', 'messages', 'messages.id = :messageId', {messageId: message.id})
+┊   ┊475┊        .getOne())!;
 ┊352┊476┊    },
-┊353┊   ┊    chat: (recipient) => chats.find(chat => recipient.chatId === chat.id)!,
 ┊354┊477┊  },
 ┊355┊478┊};
```

[}]: #

`QueryBuilder` is one of the most powerful features of `TypeORM` - it allows you to build SQL queries using elegant and convenient syntax, execute them and get automatically transformed entities.

You can find more informations on `TypeORM` on http://typeorm.io

The best part is that you won't have to do anything on the client, everything will be completely transparent to it, even if migrated from NoSQL-like db structure to a relational one!
Of course, you could remove the custom normalization for the messages because now they have their own table and they are no longer embedded (so they have unique IDs), but we could leave it as well in order to be free to use any kind of backend.

[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/tree/master@1.4.0/.tortilla/manuals/views/step15.md) |
|:----------------------|

[}]: #
