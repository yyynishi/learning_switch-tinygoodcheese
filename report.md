#レポート課題
##課題(10/21)
複数スイッチ対応版(multi_learning_switch.rb)の動作解説
###回答
単スイッチのlearning_switch.rbは起動時にフォワーディングデータベースを新規に作成する。
これに対して、multi_learning_switch.rbでは、フォワーディングデータベースを格納するための配列を用意し、読み込んだconfファイルに従ってスイッチとホストを用意する。
このように、スイッチを配列に格納することで、各スイッチからのpacket_inや、各スイッチに対するflow_mod, packet_outを判別し、実行する。

###実行結果
予め、learning_switchディレクトリ内に用意されていた複数スイッチ用のconfファイル(trema.multi.conf)を読み込んで、multi_learning_switch.rbを起動し、その上でパケットの送付を何回か行って結果を出力した。
trema.multi.confファイルの設定では、４つのスイッチがあり、各スイッチそれぞれに１つのホストが接続されている。各スイッチは互いに接続されている。

	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host1 --dest host2
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host3 --dest host2
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host4 --dest host2
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host2 --dest host3
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema show_stats host2
	Packets sent:
  	192.168.0.2 -> 192.168.0.3 = 1 packet
	Packets received:
  	192.168.0.1 -> 192.168.0.2 = 1 packet
  	192.168.0.3 -> 192.168.0.2 = 1 packet
  	192.168.0.4 -> 192.168.0.2 = 1 packet

この結果から、trema上で４つのスイッチが同時に仮想的に動作していることが分かった。


##課題(10/28)
OpenFlow1.3のスイッチの動作の解説
###回答

* 概要
    * learning_switch.rbでは１つのフローテーブルしか存在していなかったが、learning_switch13.rbでは２種類のテーブルが使用されている。以下に各テーブルのエントリの説明を示す。優先度は値が大きいほど高く、先にマッチするかの比較が発生する。

* エントリ０：INGRESS_FILTERING_TABLE
概要：フィルタリングを行うためのテーブル
    * multicast_mac_drop_flow_entry
	デフォルトのエントリである。マルチキャストするためのパケットが到着した時に、そのパケットをドロップする。優先度は2。
    * default_forwarding_flow_entry
	デフォルトのエントリである。全てのパケットがマッチし、次のテーブル(FORWARDING_TABLE)にパケットを渡す。優先度は1。

* エントリ１：FORWARDING_TABLE
概要：パケットのフォワーディングを行うためのテーブル
    * forwarding_flow_entry
	packet_inが発生すると書き込まれるエントリである。パケットの送信元のMACアドレスを記録し、そこに対するパケットを送信するためのエントリである。優先度は2。
    * default_forwarding_flow_entry
	デフォルトのエントリである。全てのパケットがマッチし、フラッディングを行う。優先度は1。




###実行結果

*起動時
tremaを起動し、パケットの送信を行わない状態では、デフォルトのエントリが各テーブルに書き込まれる。具体的には、INGRESS_FILTERING_TABLEに、
multicast_mac_drop_flow_entry(下記１つ目のエントリ)とdefault_forwarding_flow_entry(下記２つ目のエントリ)が書き込まれ、FORWARDING_TABLEにdefault_forwarding_flow_entry(下記３つ目のエントリ)が書き込まれる。

	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ sudo ovs-ofctl dump-flows brlsw --protocol=OpenFlow13OFPST_FLOW reply (OF1.3) (xid=0x2):
 	cookie=0x0, duration=1.815s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:5e:00:00:00/ff:ff:ff:00:00:00 actions=drop
 	cookie=0x0, duration=1.775s, table=0, n_packets=0, n_bytes=0, priority=1 actions=goto_table:1
 	cookie=0x0, duration=1.775s, table=1, n_packets=0, n_bytes=0, priority=1 actions=CONTROLLER:65535,FLOOD


*パケット送信時
上記の状態からhost2からhost1に対してパケットを送信し、dump-flowした時の出力を以下に示す。上記のエントリに加え、FORWARDING_TABLEに新たなエントリが追加されている。これはpacket_inが発生し、add_forwarding_flow_entryによって、host2のMACアドレスを記録し、host2に対してパケットを送信するためのエントリが書き込まれたことを示している。優先度はフラッディングを行うエントリより高いので、フラディングを行う前に、宛先がhost2かどうか確認するということが分かる。

	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host2 --dest host1
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ sudo ovs-ofctl dump-flows brlsw --protocol=OpenFlow13
	OFPST_FLOW reply (OF1.3) (xid=0x2):
 	cookie=0x0, duration=71.838s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:5e:00:00:00/ff:ff:ff:00:00:00 actions=drop
 	cookie=0x0, duration=71.798s, table=0, n_packets=1, n_bytes=42, priority=1 actions=goto_table:1
 	cookie=0x0, duration=11.691s, table=1, n_packets=0, n_bytes=0, idle_timeout=180, priority=2,dl_dst=5f:53:d4:2d:db:af actions=output:2
 	cookie=0x0, duration=71.798s, table=1, n_packets=1, n_bytes=42, priority=1 actions=CONTROLLER:65535,FLOOD