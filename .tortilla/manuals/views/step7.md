# Step 7: Messages and chats removal

[//]: # (head-end)


## Client

Since we're now familiar with the way mutations work, it's time to add messages and chats removal to our list of features!
Since the most annoying part is going to be dealing with the user interface (because a multiple selection started by a press event is involved), I created an Angular directive to ease the process.

Let's start by adding `ngx-selectable-list`:

    yarn add ngx-selectable-list

[{]: <helper> (diffStep "7.1" files="^\(?!yarn.lock$\).*" module="client")

#### [Step 7.1: Add SelectableListModule](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/abcd720)

##### Changed package.json
```diff
@@ -35,6 +35,7 @@
 ┊35┊35┊    "graphql": "^14.1.1",
 ┊36┊36┊    "graphql-tag": "^2.10.1",
 ┊37┊37┊    "hammerjs": "^2.0.8",
+┊  ┊38┊    "ngx-selectable-list": "^1.2.1",
 ┊38┊39┊    "rxjs": "~6.3.3",
 ┊39┊40┊    "tslib": "^1.9.0",
 ┊40┊41┊    "zone.js": "~0.8.26"
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;chats-lister.module.ts
```diff
@@ -10,6 +10,7 @@
 ┊10┊10┊import {ChatsComponent} from './containers/chats/chats.component';
 ┊11┊11┊import {ChatsListComponent} from './components/chats-list/chats-list.component';
 ┊12┊12┊import {SharedModule} from '../shared/shared.module';
+┊  ┊13┊import {NgxSelectableListModule} from 'ngx-selectable-list';
 ┊13┊14┊
 ┊14┊15┊const routes: Routes = [
 ┊15┊16┊  {path: '', redirectTo: 'chats', pathMatch: 'full'},
```
```diff
@@ -37,6 +38,7 @@
 ┊37┊38┊    FormsModule,
 ┊38┊39┊    // Feature modules
 ┊39┊40┊    SharedModule,
+┊  ┊41┊    NgxSelectableListModule,
 ┊40┊42┊  ],
 ┊41┊43┊  providers: [
 ┊42┊44┊    ChatsService,
```

[}]: #

Because we're going to use the `libSelectableList` directive, we can replace Outputs and EventEmitters with it:

[{]: <helper> (diffStep "7.2" files="src/app/chats-lister/components/chat-item/chat-item.component.ts, src/app/chats-lister/components/chats-list/chats-list.component.ts, src/app/chat-viewer/components/messages-list/messages-list.component.ts" module="client")

#### [Step 7.2: Remove messages and chats](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/31f9205)

##### Changed src&#x2F;app&#x2F;chat-viewer&#x2F;components&#x2F;messages-list&#x2F;messages-list.component.ts
```diff
@@ -1,14 +1,17 @@
 ┊ 1┊ 1┊import {Component, Input} from '@angular/core';
 ┊ 2┊ 2┊import {GetChat} from '../../../../graphql';
+┊  ┊ 3┊import {SelectableListDirective} from 'ngx-selectable-list';
 ┊ 3┊ 4┊
 ┊ 4┊ 5┊@Component({
 ┊ 5┊ 6┊  selector: 'app-messages-list',
 ┊ 6┊ 7┊  template: `
 ┊ 7┊ 8┊    <mat-list>
 ┊ 8┊ 9┊      <mat-list-item *ngFor="let message of messages">
-┊ 9┊  ┊        <app-message-item [item]="message" [isGroup]="isGroup"></app-message-item>
+┊  ┊10┊        <app-message-item [item]="message" [isGroup]="isGroup"
+┊  ┊11┊                          libSelectableItem></app-message-item>
 ┊10┊12┊      </mat-list-item>
 ┊11┊13┊    </mat-list>
+┊  ┊14┊    <ng-content *ngIf="selectableListDirective.selecting"></ng-content>
 ┊12┊15┊  `,
 ┊13┊16┊  styleUrls: ['messages-list.component.scss'],
 ┊14┊17┊})
```
```diff
@@ -20,5 +23,5 @@
 ┊20┊23┊  @Input()
 ┊21┊24┊  isGroup: boolean;
 ┊22┊25┊
-┊23┊  ┊  constructor() {}
+┊  ┊26┊  constructor(public selectableListDirective: SelectableListDirective) {}
 ┊24┊27┊}
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;components&#x2F;chat-item&#x2F;chat-item.component.ts
```diff
@@ -4,7 +4,7 @@
 ┊ 4┊ 4┊@Component({
 ┊ 5┊ 5┊  selector: 'app-chat-item',
 ┊ 6┊ 6┊  template: `
-┊ 7┊  ┊    <div class="chat-row" (click)="selectChat()">
+┊  ┊ 7┊    <div class="chat-row">
 ┊ 8┊ 8┊      <img class="chat-pic" [src]="chat.picture || 'assets/default-profile-pic.jpg'">
 ┊ 9┊ 9┊      <div class="chat-info">
 ┊10┊10┊        <div class="chat-name">{{ chat.name }}</div>
```
```diff
@@ -19,11 +19,4 @@
 ┊19┊19┊  // tslint:disable-next-line:no-input-rename
 ┊20┊20┊  @Input('item')
 ┊21┊21┊  chat: GetChats.Chats;
-┊22┊  ┊
-┊23┊  ┊  @Output()
-┊24┊  ┊  select = new EventEmitter<string>();
-┊25┊  ┊
-┊26┊  ┊  selectChat() {
-┊27┊  ┊    this.select.emit(this.chat.id);
-┊28┊  ┊  }
 ┊29┊22┊}
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;components&#x2F;chats-list&#x2F;chats-list.component.ts
```diff
@@ -1,14 +1,17 @@
-┊ 1┊  ┊import {Component, EventEmitter, Input, Output} from '@angular/core';
+┊  ┊ 1┊import {Component, Input} from '@angular/core';
 ┊ 2┊ 2┊import {GetChats} from '../../../../graphql';
+┊  ┊ 3┊import {SelectableListDirective} from 'ngx-selectable-list';
 ┊ 3┊ 4┊
 ┊ 4┊ 5┊@Component({
 ┊ 5┊ 6┊  selector: 'app-chats-list',
 ┊ 6┊ 7┊  template: `
 ┊ 7┊ 8┊    <mat-list>
 ┊ 8┊ 9┊      <mat-list-item *ngFor="let chat of chats">
-┊ 9┊  ┊        <app-chat-item [item]="chat" (select)="selectChat($event)"></app-chat-item>
+┊  ┊10┊        <app-chat-item [item]="chat"
+┊  ┊11┊                       libSelectableItem></app-chat-item>
 ┊10┊12┊      </mat-list-item>
 ┊11┊13┊    </mat-list>
+┊  ┊14┊    <ng-content *ngIf="selectableListDirective.selecting"></ng-content>
 ┊12┊15┊  `,
 ┊13┊16┊  styleUrls: ['chats-list.component.scss'],
 ┊14┊17┊})
```
```diff
@@ -17,12 +20,5 @@
 ┊17┊20┊  @Input('items')
 ┊18┊21┊  chats: GetChats.Chats[];
 ┊19┊22┊
-┊20┊  ┊  @Output()
-┊21┊  ┊  select = new EventEmitter<string>();
-┊22┊  ┊
-┊23┊  ┊  constructor() {}
-┊24┊  ┊
-┊25┊  ┊  selectChat(id: string) {
-┊26┊  ┊    this.select.emit(id);
-┊27┊  ┊  }
+┊  ┊23┊  constructor(public selectableListDirective: SelectableListDirective) {}
 ┊28┊24┊}
```

[}]: #

Don't forget to add the `NgxSelectableListModule` to `ChatViewer` and `ChatLister` modules.

We also created a `ConfirmSelectionComponent` to use for content projection, since our selectable list directive will be able to listen to its events.

[{]: <helper> (diffStep "7.2" files="src/app/shared/components/confirm-selection/confirm-selection.component.scss, src/app/shared/components/confirm-selection/confirm-selection.component.ts, src/app/shared/shared.module.ts" module="client")

#### [Step 7.2: Remove messages and chats](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/31f9205)

##### Added src&#x2F;app&#x2F;shared&#x2F;components&#x2F;confirm-selection&#x2F;confirm-selection.component.scss
```diff
@@ -0,0 +1,6 @@
+┊ ┊1┊:host {
+┊ ┊2┊  display: block;
+┊ ┊3┊  position: absolute;
+┊ ┊4┊  bottom: 5vw;
+┊ ┊5┊  right: 5vw;
+┊ ┊6┊}
```

##### Added src&#x2F;app&#x2F;shared&#x2F;components&#x2F;confirm-selection&#x2F;confirm-selection.component.ts
```diff
@@ -0,0 +1,21 @@
+┊  ┊ 1┊import {Component, EventEmitter, Input, Output} from '@angular/core';
+┊  ┊ 2┊
+┊  ┊ 3┊@Component({
+┊  ┊ 4┊  selector: 'app-confirm-selection',
+┊  ┊ 5┊  template: `
+┊  ┊ 6┊    <button mat-fab color="secondary" (click)="handleClick()">
+┊  ┊ 7┊      <mat-icon aria-label="Icon-button">{{ icon }}</mat-icon>
+┊  ┊ 8┊    </button>
+┊  ┊ 9┊  `,
+┊  ┊10┊  styleUrls: ['./confirm-selection.component.scss'],
+┊  ┊11┊})
+┊  ┊12┊export class ConfirmSelectionComponent {
+┊  ┊13┊  @Input()
+┊  ┊14┊  icon = 'delete';
+┊  ┊15┊  @Output()
+┊  ┊16┊  emitClick = new EventEmitter<null>();
+┊  ┊17┊
+┊  ┊18┊  handleClick() {
+┊  ┊19┊    this.emitClick.emit();
+┊  ┊20┊  }
+┊  ┊21┊}
```

##### Changed src&#x2F;app&#x2F;shared&#x2F;shared.module.ts
```diff
@@ -1,19 +1,23 @@
 ┊ 1┊ 1┊import {BrowserModule} from '@angular/platform-browser';
 ┊ 2┊ 2┊import {NgModule} from '@angular/core';
 ┊ 3┊ 3┊
-┊ 4┊  ┊import {MatToolbarModule} from '@angular/material';
+┊  ┊ 4┊import {MatButtonModule, MatIconModule, MatToolbarModule} from '@angular/material';
 ┊ 5┊ 5┊import {ToolbarComponent} from './components/toolbar/toolbar.component';
 ┊ 6┊ 6┊import {FormsModule} from '@angular/forms';
 ┊ 7┊ 7┊import {BrowserAnimationsModule} from '@angular/platform-browser/animations';
+┊  ┊ 8┊import {ConfirmSelectionComponent} from './components/confirm-selection/confirm-selection.component';
 ┊ 8┊ 9┊
 ┊ 9┊10┊@NgModule({
 ┊10┊11┊  declarations: [
 ┊11┊12┊    ToolbarComponent,
+┊  ┊13┊    ConfirmSelectionComponent,
 ┊12┊14┊  ],
 ┊13┊15┊  imports: [
 ┊14┊16┊    BrowserModule,
 ┊15┊17┊    // Material
 ┊16┊18┊    MatToolbarModule,
+┊  ┊19┊    MatIconModule,
+┊  ┊20┊    MatButtonModule,
 ┊17┊21┊    // Animations
 ┊18┊22┊    BrowserAnimationsModule,
 ┊19┊23┊    // Forms
```
```diff
@@ -22,6 +26,7 @@
 ┊22┊26┊  providers: [],
 ┊23┊27┊  exports: [
 ┊24┊28┊    ToolbarComponent,
+┊  ┊29┊    ConfirmSelectionComponent,
 ┊25┊30┊  ],
 ┊26┊31┊})
 ┊27┊32┊export class SharedModule {
```

[}]: #

Great, let's finish the component part by adding the connection between our container components and the `ChatsService`. We're going to use `removeMessages` and `removeChat` methods.

[{]: <helper> (diffStep "7.2" files="src/app/chat-viewer/containers/chat/chat.component.ts, src/app/chat-viewer/containers/chat/chat.component.spec.ts, src/app/chats-lister/containers/chats/chats.component.ts, src/app/chats-lister/containers/chats/chats.component.spec.ts" module="client")

#### [Step 7.2: Remove messages and chats](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/31f9205)

##### Changed src&#x2F;app&#x2F;chat-viewer&#x2F;containers&#x2F;chat&#x2F;chat.component.spec.ts
```diff
@@ -20,6 +20,7 @@
 ┊20┊20┊  APOLLO_TESTING_CACHE,
 ┊21┊21┊} from 'apollo-angular/testing';
 ┊22┊22┊import { InMemoryCache } from 'apollo-cache-inmemory';
+┊  ┊23┊import { NgxSelectableListModule } from 'ngx-selectable-list';
 ┊23┊24┊
 ┊24┊25┊import { dataIdFromObject } from '../../../graphql.module';
 ┊25┊26┊import { ChatComponent } from './chat.component';
```
```diff
@@ -116,6 +117,7 @@
 ┊116┊117┊        SharedModule,
 ┊117┊118┊        ApolloTestingModule,
 ┊118┊119┊        RouterTestingModule,
+┊   ┊120┊        NgxSelectableListModule,
 ┊119┊121┊      ],
 ┊120┊122┊      providers: [
 ┊121┊123┊        ChatsService,
```

##### Changed src&#x2F;app&#x2F;chat-viewer&#x2F;containers&#x2F;chat&#x2F;chat.component.ts
```diff
@@ -13,7 +13,10 @@
 ┊13┊13┊      <div class="title">{{ name }}</div>
 ┊14┊14┊    </app-toolbar>
 ┊15┊15┊    <div class="container">
-┊16┊  ┊      <app-messages-list [items]="messages" [isGroup]="isGroup"></app-messages-list>
+┊  ┊16┊      <app-messages-list [items]="messages" [isGroup]="isGroup"
+┊  ┊17┊                         libSelectableList="multiple_press" (multiple)="deleteMessages($event)">
+┊  ┊18┊        <app-confirm-selection #confirmSelection></app-confirm-selection>
+┊  ┊19┊      </app-messages-list>
 ┊17┊20┊      <app-new-message (newMessage)="addMessage($event)"></app-new-message>
 ┊18┊21┊    </div>
 ┊19┊22┊  `,
```
```diff
@@ -48,4 +51,8 @@
 ┊48┊51┊  addMessage(content: string) {
 ┊49┊52┊    this.chatsService.addMessage(this.chatId, content).subscribe();
 ┊50┊53┊  }
+┊  ┊54┊
+┊  ┊55┊  deleteMessages(messageIds: string[]) {
+┊  ┊56┊    this.chatsService.removeMessages(this.chatId, this.messages, messageIds).subscribe();
+┊  ┊57┊  }
 ┊51┊58┊}
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;containers&#x2F;chats&#x2F;chats.component.spec.ts
```diff
@@ -15,6 +15,7 @@
 ┊15┊15┊import { InMemoryCache } from 'apollo-cache-inmemory';
 ┊16┊16┊import { By } from '@angular/platform-browser';
 ┊17┊17┊import { RouterTestingModule } from '@angular/router/testing';
+┊  ┊18┊import { NgxSelectableListModule } from 'ngx-selectable-list';
 ┊18┊19┊
 ┊19┊20┊import { GetChats } from '../../../../graphql';
 ┊20┊21┊import { dataIdFromObject } from '../../../graphql.module';
```
```diff
@@ -332,6 +333,7 @@
 ┊332┊333┊        MatListModule,
 ┊333┊334┊        ApolloTestingModule,
 ┊334┊335┊        RouterTestingModule,
+┊   ┊336┊        NgxSelectableListModule,
 ┊335┊337┊      ],
 ┊336┊338┊      providers: [
 ┊337┊339┊        ChatsService,
```

##### Changed src&#x2F;app&#x2F;chats-lister&#x2F;containers&#x2F;chats&#x2F;chats.component.ts
```diff
@@ -28,9 +28,13 @@
 ┊28┊28┊      </button>
 ┊29┊29┊    </mat-menu>
 ┊30┊30┊
-┊31┊  ┊    <app-chats-list [items]="chats$ | async" (select)="goToChat($event)"></app-chats-list>
+┊  ┊31┊    <app-chats-list [items]="chats$ | async"
+┊  ┊32┊                    libSelectableList="both"
+┊  ┊33┊                    (single)="goToChat($event)" (multiple)="deleteChats($event)" (isSelecting)="isSelecting = $event">
+┊  ┊34┊      <app-confirm-selection #confirmSelection></app-confirm-selection>
+┊  ┊35┊    </app-chats-list>
 ┊32┊36┊
-┊33┊  ┊    <button class="chat-button" mat-fab color="secondary">
+┊  ┊37┊    <button *ngIf="!isSelecting" class="chat-button" mat-fab color="secondary">
 ┊34┊38┊      <mat-icon aria-label="Icon-button with a + icon">chat</mat-icon>
 ┊35┊39┊    </button>
 ┊36┊40┊  `,
```
```diff
@@ -38,6 +42,7 @@
 ┊38┊42┊})
 ┊39┊43┊export class ChatsComponent implements OnInit {
 ┊40┊44┊  chats$: Observable<GetChats.Chats[]>;
+┊  ┊45┊  isSelecting = false;
 ┊41┊46┊
 ┊42┊47┊  constructor(private chatsService: ChatsService,
 ┊43┊48┊              private router: Router) {
```
```diff
@@ -50,4 +55,10 @@
 ┊50┊55┊  goToChat(chatId: string) {
 ┊51┊56┊    this.router.navigate(['/chat', chatId]);
 ┊52┊57┊  }
+┊  ┊58┊
+┊  ┊59┊  deleteChats(chatIds: string[]) {
+┊  ┊60┊    chatIds.forEach(chatId => {
+┊  ┊61┊      this.chatsService.removeChat(chatId).subscribe();
+┊  ┊62┊    });
+┊  ┊63┊  }
 ┊53┊64┊}
```

[}]: #

### Connecting actions with the server

The UI part is pretty much complete but we still need to implement two methods in the `ChatsService`. We're going to create mutations to remove a chat, selected messages or all of them.

[{]: <helper> (diffStep "7.2" files="src/graphql/removeChat.mutation.ts, src/graphql/removeMessages.mutation.ts, src/graphql/removeAllMessages.mutation.ts" module="client")

#### [Step 7.2: Remove messages and chats](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/31f9205)

##### Added src&#x2F;graphql&#x2F;removeAllMessages.mutation.ts
```diff
@@ -0,0 +1,8 @@
+┊ ┊1┊import gql from 'graphql-tag';
+┊ ┊2┊
+┊ ┊3┊// We use the gql tag to parse our query string into a query document
+┊ ┊4┊export const removeAllMessagesMutation = gql`
+┊ ┊5┊  mutation RemoveAllMessages($chatId: ID!, $all: Boolean) {
+┊ ┊6┊    removeMessages(chatId: $chatId, all: $all)
+┊ ┊7┊  }
+┊ ┊8┊`;
```

##### Added src&#x2F;graphql&#x2F;removeChat.mutation.ts
```diff
@@ -0,0 +1,8 @@
+┊ ┊1┊import gql from 'graphql-tag';
+┊ ┊2┊
+┊ ┊3┊// We use the gql tag to parse our query string into a query document
+┊ ┊4┊export const removeChatMutation = gql`
+┊ ┊5┊  mutation RemoveChat($chatId: ID!) {
+┊ ┊6┊    removeChat(chatId: $chatId)
+┊ ┊7┊  }
+┊ ┊8┊`;
```

##### Added src&#x2F;graphql&#x2F;removeMessages.mutation.ts
```diff
@@ -0,0 +1,8 @@
+┊ ┊1┊import gql from 'graphql-tag';
+┊ ┊2┊
+┊ ┊3┊// We use the gql tag to parse our query string into a query document
+┊ ┊4┊export const removeMessagesMutation = gql`
+┊ ┊5┊  mutation RemoveMessages($chatId: ID!, $messageIds: [ID!]) {
+┊ ┊6┊    removeMessages(chatId: $chatId, messageIds: $messageIds)
+┊ ┊7┊  }
+┊ ┊8┊`;
```

[}]: #

Once again, we run the GraphQL Code Generator to get the GQL services:

    yarn generator

Now we can import those services, create `removeChat` and `removeMessages` methods in which we call a mutation.

[{]: <helper> (diffStep "7.2" files="src/app/services/chats.service.ts" module="client")

#### [Step 7.2: Remove messages and chats](https://github.com/Urigo/WhatsApp-Clone-Client-Angular/commit/31f9205)

##### Changed src&#x2F;app&#x2F;services&#x2F;chats.service.ts
```diff
@@ -4,10 +4,16 @@
 ┊ 4┊ 4┊  GetChatsGQL,
 ┊ 5┊ 5┊  GetChatGQL,
 ┊ 6┊ 6┊  AddMessageGQL,
+┊  ┊ 7┊  RemoveChatGQL,
+┊  ┊ 8┊  RemoveMessagesGQL,
+┊  ┊ 9┊  RemoveAllMessagesGQL,
 ┊ 7┊10┊  AddMessage,
 ┊ 8┊11┊  GetChats,
-┊ 9┊  ┊  GetChat
+┊  ┊12┊  GetChat,
+┊  ┊13┊  RemoveMessages,
+┊  ┊14┊  RemoveAllMessages,
 ┊10┊15┊} from '../../graphql';
+┊  ┊16┊import { DataProxy } from 'apollo-cache';
 ┊11┊17┊
 ┊12┊18┊@Injectable()
 ┊13┊19┊export class ChatsService {
```
```diff
@@ -16,7 +22,10 @@
 ┊16┊22┊  constructor(
 ┊17┊23┊    private getChatsGQL: GetChatsGQL,
 ┊18┊24┊    private getChatGQL: GetChatGQL,
-┊19┊  ┊    private addMessageGQL: AddMessageGQL
+┊  ┊25┊    private addMessageGQL: AddMessageGQL,
+┊  ┊26┊    private removeChatGQL: RemoveChatGQL,
+┊  ┊27┊    private removeMessagesGQL: RemoveMessagesGQL,
+┊  ┊28┊    private removeAllMessagesGQL: RemoveAllMessagesGQL,
 ┊20┊29┊  ) {}
 ┊21┊30┊
 ┊22┊31┊  getChats() {
```
```diff
@@ -92,4 +101,111 @@
 ┊ 92┊101┊      }
 ┊ 93┊102┊    });
 ┊ 94┊103┊  }
+┊   ┊104┊
+┊   ┊105┊  removeChat(chatId: string) {
+┊   ┊106┊    return this.removeChatGQL.mutate(
+┊   ┊107┊      {
+┊   ┊108┊        chatId,
+┊   ┊109┊      }, {
+┊   ┊110┊        update: (store, { data: { removeChat } }) => {
+┊   ┊111┊          // Read the data from our cache for this query.
+┊   ┊112┊          const {chats} = store.readQuery<GetChats.Query, GetChats.Variables>({
+┊   ┊113┊            query: this.getChatsGQL.document,
+┊   ┊114┊            variables: {
+┊   ┊115┊              amount: this.messagesAmount,
+┊   ┊116┊            },
+┊   ┊117┊          });
+┊   ┊118┊          // Remove the chat (mutable)
+┊   ┊119┊          for (const index of chats.keys()) {
+┊   ┊120┊            if (chats[index].id === removeChat) {
+┊   ┊121┊              chats.splice(index, 1);
+┊   ┊122┊            }
+┊   ┊123┊          }
+┊   ┊124┊          // Write our data back to the cache.
+┊   ┊125┊          store.writeQuery<GetChats.Query, GetChats.Variables>({
+┊   ┊126┊            query: this.getChatsGQL.document,
+┊   ┊127┊            variables: {
+┊   ┊128┊              amount: this.messagesAmount,
+┊   ┊129┊            },
+┊   ┊130┊            data: {
+┊   ┊131┊              chats,
+┊   ┊132┊            },
+┊   ┊133┊          });
+┊   ┊134┊        },
+┊   ┊135┊      }
+┊   ┊136┊    );
+┊   ┊137┊  }
+┊   ┊138┊
+┊   ┊139┊  removeMessages(chatId: string, messages: GetChat.Messages[], messageIdsOrAll: string[] | boolean) {
+┊   ┊140┊    let ids: string[] = [];
+┊   ┊141┊
+┊   ┊142┊    const options = {
+┊   ┊143┊      update: (store: DataProxy, { data: { removeMessages } }: {data: RemoveMessages.Mutation | RemoveAllMessages.Mutation}) => {
+┊   ┊144┊        // Update the messages cache
+┊   ┊145┊        {
+┊   ┊146┊          // Read the data from our cache for this query.
+┊   ┊147┊          const {chat} = store.readQuery<GetChat.Query, GetChat.Variables>({
+┊   ┊148┊            query: this.getChatGQL.document,
+┊   ┊149┊            variables: {
+┊   ┊150┊              chatId,
+┊   ┊151┊            }
+┊   ┊152┊          });
+┊   ┊153┊          // Remove the messages (mutable)
+┊   ┊154┊          removeMessages.forEach(messageId => {
+┊   ┊155┊            for (const index of chat.messages.keys()) {
+┊   ┊156┊              if (chat.messages[index].id === messageId) {
+┊   ┊157┊                chat.messages.splice(index, 1);
+┊   ┊158┊              }
+┊   ┊159┊            }
+┊   ┊160┊          });
+┊   ┊161┊          // Write our data back to the cache.
+┊   ┊162┊          store.writeQuery<GetChat.Query, GetChat.Variables>({
+┊   ┊163┊            query: this.getChatGQL.document,
+┊   ┊164┊            data: {
+┊   ┊165┊              chat
+┊   ┊166┊            }
+┊   ┊167┊          });
+┊   ┊168┊        }
+┊   ┊169┊        // Update last message cache
+┊   ┊170┊        {
+┊   ┊171┊          // Read the data from our cache for this query.
+┊   ┊172┊          const {chats} = store.readQuery<GetChats.Query, GetChats.Variables>({
+┊   ┊173┊            query: this.getChatsGQL.document,
+┊   ┊174┊            variables: {
+┊   ┊175┊              amount: this.messagesAmount,
+┊   ┊176┊            },
+┊   ┊177┊          });
+┊   ┊178┊          // Fix last message
+┊   ┊179┊          chats.find(chat => chat.id === chatId).messages = messages
+┊   ┊180┊            .filter(message => !ids.includes(message.id));
+┊   ┊181┊          // Write our data back to the cache.
+┊   ┊182┊          store.writeQuery<GetChats.Query, GetChats.Variables>({
+┊   ┊183┊            query: this.getChatsGQL.document,
+┊   ┊184┊            variables: {
+┊   ┊185┊              amount: this.messagesAmount,
+┊   ┊186┊            },
+┊   ┊187┊            data: {
+┊   ┊188┊              chats,
+┊   ┊189┊            },
+┊   ┊190┊          });
+┊   ┊191┊        }
+┊   ┊192┊      }
+┊   ┊193┊    };
+┊   ┊194┊
+┊   ┊195┊    if (typeof messageIdsOrAll === 'boolean') {
+┊   ┊196┊      ids = messages.map(message => message.id);
+┊   ┊197┊
+┊   ┊198┊      return this.removeAllMessagesGQL.mutate({
+┊   ┊199┊        chatId,
+┊   ┊200┊        all: messageIdsOrAll
+┊   ┊201┊      }, options);
+┊   ┊202┊    } else {
+┊   ┊203┊      ids = messageIdsOrAll;
+┊   ┊204┊
+┊   ┊205┊      return this.removeMessagesGQL.mutate({
+┊   ┊206┊        chatId,
+┊   ┊207┊        messageIds: messageIdsOrAll,
+┊   ┊208┊      }, options);
+┊   ┊209┊    }
+┊   ┊210┊  }
 ┊ 95┊211┊}
```

[}]: #

It might seem like a lot but we simply just updating the store there.

### Summary

The selectable list directive supports much more different use cases, for info please read the documentation.

As you can see `ngx-selectable-list` takes care of most of the boilerplate, giving us the freedom to concentrate on the actual code.


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/tree/master@2.0.0/.tortilla/manuals/views/step6.md) | [Next Step >](https://github.com/Urigo/whatsapp-textrepo-angularcli-express/tree/master@2.0.0/.tortilla/manuals/views/step8.md) |
|:--------------------------------|--------------------------------:|

[}]: #
