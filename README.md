## 参考サイト

ほぼ以下のサイトを参考にしました。
http://qiita.com/jnchito/items/aec75fab42804287d71b

変更点のみ以下記載。

[1]
application.jsにjQueryがrequireされているかを確認

[2]
app/assets/javascripts/cable.coffeeの以下の記載の追加は不要
@App ||= {}
App.cable = ActionCable.createConsumer()

	