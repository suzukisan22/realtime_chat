## 参考サイト

ほぼ以下のサイトを参考にしました。
http://qiita.com/jnchito/items/aec75fab42804287d71b

変更点のみ以下記載。

#1

application.jsにjQueryがrequireされているかを確認

#2 

app/assets/javascripts/cable.coffeeの以下の記載の追加は不要
@App ||= {}
App.cable = ActionCable.createConsumer()

#3

routeの設定が不要
`mount ActionCable.server => '/cable'`


# action cableについて


## action cableとは
rails 5の目玉機能であるAction Cableは、Rails上でWebSocketによる双方向通信を簡単に実現する新しい機能です。

## webSocketとは
WebSocket（ウェブソケット）とは、サーバーを介してWebブラウザ間で双方向通信を行う仕組みです

## Action Cable
### 機能の整理

#### コネクション
	- 機能概要
		- WebSocket通信をサーバーとクライアントで接続する仕組み

	- 実装の配置箇所
		- app/channels/application_cable/connection.rbとして提供

#### チャネル
	- 機能概要
		- WebSocketを使う上で必要となるサーバー側の仕組み

	- 実装の配置箇所
		- app/channels配下にApplicationCable::Channelを継承して利用

#### クライアント
	- 機能概要
		- WebSocketを使う上で必要となるクライアント側の仕組み。JavaScript/CoffeeScriptで提供

	- 実装の配置箇所
		- app/assets/javascripts/channels配下に配置

#### ジェネレーター
	- 機能概要
		- rails g channelコマンドによるチャネル・クライアントのコードひな型自動生成

## ソース解説
### app/assets/javascripts/cable.js

```//= require action_cable
//= require_self
//= require_tree ./channels
(function() {
  this.App || (this.App = {});

  App.cable = ActionCable.createConsumer();

}).call(this);
```


- 「cable.js」は、クライアントからサーバーに対してWebSocket接続しているコードです。App.cable = ActionCable.createConsumer();でWebSocket通信を確立しています。

### app/assets/javascripts/channels/room.coffee
- 

```
App.room = App.cable.subscriptions.create "RoomChannel",

…（中略）…

speak: (message) ->

    @perform 'speak', message: message

```

- App.chat_messageの定義の1行目で、Action Cableのサーバー側のチャネルをcreateしています。

- createの引数にChatMessageChannelが指定されている。よって、「app/channels/room_channel.rb」で指定されるサーバー側のチャネルにクライアント側から接続します。

- デフォルトのspeakメソッドを書き換え、引数に指定されたmessageをサーバー側のチャネルのspeakメソッドに引数として渡すようにします。

- @perform 'speak', message: messageと記述すると、RoomChannelのspeakメソッドを呼び出すことができます。

- このCoffeeScriptコードを定義したspeakメソッドをクライアントから呼び出すには、App.room.speak（発言メッセージ）とします。これでクライアント側から発言メッセージをサーバー側に送ることができます。

### app/channels/room_channel.rb
- 

```…（中略）…
class ChatMessageChannel < ApplicationCable::Channel
  def subscribed
    stream_from 'chat_message_channel'
  end
…（中略）…
  def speak(data)
    ActionCable.server.broadcast 'chat_message_channel', message: data['message']
  end
end
```

- speakメソッドはクライアント側で、発言メッセージを指定したキーワード引数を取るように定義しました。サーバー側では引数に指定したdata経由で、data['message']とすることで発言メッセージを取り出すことができます。

- speakメソッドのActionCable.server.broadcastでは第1引数にチャネル名を、第2引数に発言メッセージを指定することで、サーバー側からRoomChannelにWebSocketで接続している全クライアントに対して発言メッセージを配信することができます。

- なおsubscribedメソッドは、各クライアントに配信する内容をどこに配信するかを定義しています。この機能はストリームと呼ばれ、Railsが提供する_stream_from_メソッドを通じて、発言メッセージを_room_channel_に接続したクライアントに配信できるようになります。
	
### app/assets/javascripts/channels/room.coffee
- 

```received: (data) ->
    $('#messages').append data['message']
```

- サーバー側のチャネルからブロードキャストされた発言メッセージをクライアント側で受け取る処理は、receivedメソッドに記述します。

- サーバー側から送られてきたデータを引数dataで受け取ります。発言メッセージはdata['message']で取り出すことができます。



## 実行コマンド
- cmd > rails 5.1 new campfire --skip-spring

- cmd > cd campfire

- cmd > rails g controller rooms show

- cmd > rails g model message content:text

- cmd > rails db:migrate

- cmd > rails g channel room speak

- cmd > redis-server

- cmd > rails g job MessageBroadcast

