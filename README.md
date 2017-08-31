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
```mount ActionCable.server => '/cable'```


↓log
RoomChannel#speak({"message"=>"できた"})
   (0.1ms)  BEGIN
  SQL (0.8ms)  INSERT INTO "messages" ("content", "created_at", "updated_at") VALUES ($1, $2, $3) RETURNING "id"  [["content", "できた"], ["created_at", "2017-08-31 14:10:10.876936"], ["updated_at", "2017-08-31 14:10:10.876936"]]
   (8.6ms)  COMMIT
[ActiveJob] Enqueued MessageBroadcastJob (Job ID: af624887-c526-4bd3-80d3-fcd0dbcd215f) to Async(default) with arguments: #<GlobalID:0x007fb25f0d2ed8 @uri=#<URI::GID gid://campfire/Message/9>>
  Message Load (0.4ms)  SELECT  "messages".* FROM "messages" WHERE "messages"."id" = $1 LIMIT $2  [["id", 9], ["LIMIT", 1]]
[ActiveJob] [MessageBroadcastJob] [af624887-c526-4bd3-80d3-fcd0dbcd215f] Performing MessageBroadcastJob (Job ID: af624887-c526-4bd3-80d3-fcd0dbcd215f) from Async(default) with arguments: #<GlobalID:0x007fb25f092a90 @uri=#<URI::GID gid://campfire/Message/9>>
[ActiveJob] [MessageBroadcastJob] [af624887-c526-4bd3-80d3-fcd0dbcd215f]   Rendered messages/_message.html.erb (0.5ms)
[ActiveJob] [MessageBroadcastJob] [af624887-c526-4bd3-80d3-fcd0dbcd215f] [ActionCable] Broadcasting to room_channel: {:message=>"<div class=\"message\">\n  <p>できた</p>\n</div>\n"}
[ActiveJob] [MessageBroadcastJob] [af624887-c526-4bd3-80d3-fcd0dbcd215f] Performed MessageBroadcastJob (Job ID: af624887-c526-4bd3-80d3-fcd0dbcd215f) from Async(default) in 17.22ms
RoomChannel transmitting {"message"=>"<div class=\"message\">\n  <p>できた</p>\n</div>\n"} (via streamed from room_channel)
	
