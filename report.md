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
