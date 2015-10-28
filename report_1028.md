#レポート課題(10/28)
OpenFlow1.3のスイッチの動作の解説
##回答

* 概要
    * learning_switch.rbでは１つのフローテーブルしか存在していなかったが、learning_switch13.rbでは２種類のテーブルが使用されている。以下に各テーブルのエントリの説明を示す。優先度は値が大きいほど高く、先にマッチするかの比較が発生する。

* エントリ０：INGRESS_FILTERING_TABLE (フィルタリングを行うためのテーブル)
    * multicast_mac_drop_flow_entry
	デフォルトのエントリである。マルチキャストするためのパケットが到着した時に、そのパケットをドロップする。優先度は2。
    * default_forwarding_flow_entry
	デフォルトのエントリである。全てのパケットがマッチし、次のテーブル(FORWARDING_TABLE)にパケットを渡す。優先度は1。

* エントリ１：FORWARDING_TABLE (パケットのフォワーディングを行うためのテーブル)
    * forwarding_flow_entry
	packet_inが発生すると書き込まれるエントリである。パケットの送信元のMACアドレスを記録し、そこに対するパケットを送信するためのエントリである。優先度は2。
    * default_forwarding_flow_entry
	デフォルトのエントリである。全てのパケットがマッチし、フラッディングを行う。優先度は1。




##実行結果

*起動時
tremaを起動し、パケットの送信を行わない状態では、デフォルトのエントリが各テーブルに書き込まれる。具体的には、INGRESS_FILTERING_TABLEに、
multicast_mac_drop_flow_entry(下記１つ目のエントリ)とdefault_forwarding_flow_entry(下記２つ目のエントリ)が書き込まれ、FORWARDING_TABLEにdefault_forwarding_flow_entry(下記３つ目のエントリ)が書き込まれる。

	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ sudo ovs-ofctl dump-flows brlsw --protocol=OpenFlow13OFPST_FLOW reply (OF1.3) (xid=0x2):
 	cookie=0x0, duration=1.815s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:5e:00:00:00/ff:ff:ff:00:00:00 actions=drop
 	cookie=0x0, duration=1.775s, table=0, n_packets=0, n_bytes=0, priority=1 actions=goto_table:1
 	cookie=0x0, duration=1.775s, table=1, n_packets=0, n_bytes=0, priority=1 actions=CONTROLLER:65535,FLOOD


*パケット送信時
上記の状態からhost2からhost1に対してパケットを送信し、dump-flowした時の出力を以下に示す。上記のエントリに加え、FORWARDING_TABLEに新たなエントリが追加されている。これはpacket_inが発生し、add_forwarding_flow_entryによって、host2のMACアドレスを記録し、host2に対してパケットを送信するためのエントリ(下記３つ目のエントリ)が書き込まれたことを示している。優先度はフラッディングを行うエントリより高いので、フラディングを行う前に、宛先がhost2かどうか確認するということが分かる。

	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ ./bin/trema send_packets --source host2 --dest host1
	ensyuu2@ensyuu2-VirtualBox:~/Documents/learning_switch-tinygoodcheese$ sudo ovs-ofctl dump-flows brlsw --protocol=OpenFlow13
	OFPST_FLOW reply (OF1.3) (xid=0x2):
 	cookie=0x0, duration=71.838s, table=0, n_packets=0, n_bytes=0, priority=2,dl_dst=01:00:5e:00:00:00/ff:ff:ff:00:00:00 actions=drop
 	cookie=0x0, duration=71.798s, table=0, n_packets=1, n_bytes=42, priority=1 actions=goto_table:1
 	cookie=0x0, duration=11.691s, table=1, n_packets=0, n_bytes=0, idle_timeout=180, priority=2,dl_dst=5f:53:d4:2d:db:af actions=output:2
 	cookie=0x0, duration=71.798s, table=1, n_packets=1, n_bytes=42, priority=1 actions=CONTROLLER:65535,FLOOD