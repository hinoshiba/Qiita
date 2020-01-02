Lineトークテキスト化ツール
==================

* https://qiita.com/hinoshiba/items/d00313a691e317d01d7c

# やったこと
 * python LineAnalyzer.py 
 * 上を実行すると、PC内のiphoneのバックアップ選べやとでてくるので、選択
 * 含まれているlineのバックアップファイルがcsvになるぜ！！

# 動機
 * pythonの勉強がしたいから何か作ってみるものが欲しい
 * そんなに難しくない何かが良い
 * ブラウジングしていたらiPhone アナライザーの発見
  * そういえばlineのトークはたまーにバックアップを漁ることがあるなぁ
 * 教材はこれだ！

# 出来上がったもの
 * 詳細は以下
  * https://github.com/hinoshiba/LineAnalyzer

# 使用感

* 動作バージョン

```bash
User$ python --version
Python 2.7.13
```

* 起動

```bash
User$ python LineAnalyzer.py
[MSG]StartScript
[MSG]======= Choose Backup file =======
[MSG]filename:.DS_Store
[MSG]filename:45b4c87092------------b7b4d256e7f03
[MSG]filename:7b818-------------------------
which do you want >> 45b4c87092------------b7b4d256e7f03 ##ファイル名を入力する
[MSG]Target file is /Users/User/Library/Application Support/MobileSync/Backup/45b4c87092------------b7b4d256e7f03//0*/0494a***********10066a49dcbc23
[MSG]Will Get Sql_Users
[MSG]Will Get Sql_Users....Done
[MSG]Will Get Sql_Messages
[MSG]Will Get Sql_Messages....Done
[MSG]EndScript
```

* 実行結果で以下のディレクトリとcsvたちができる

```bash
User$ ls
45b4c87092------------b7b4d256e7f03
User$ ls 45b4c87092------------b7b4d256e7f03/
00002.csv       00009.csv       00015.csv       00028.csv       00041.csv       00053.csv       00066.csv       00091.csv       00100.csv       00129.csv       00142.csv
00003.csv       00010.csv       00022.csv       00033.csv       00045.csv       00054.csv       00067.csv       00093.csv       00108.csv       00133.csv       00145.csv
00007.csv       00011.csv       00023.csv       00036.csv       00051.csv       00056.csv       00078.csv       00096.csv       00110.csv       00136.csv       zchat.csv
00008.csv       00013.csv       00025.csv       00038.csv       00052.csv       00060.csv       00081.csv       00099.csv       00125.csv       00141.csv
```

* 以下のような感じで時刻とユーザとトーク内容が出力される

```
User$ head -n 2 00002.csv 
2017-08-06 15:18:08,ME,今から帰るから
2017-08-06 15:18:16,ME,16時にはつくよ！
```

* 一緒にできあがるzchat.csv
 * 全てのトークを並べた時に古い順に発言されたルームIDを並べたファイル
 * つまり、uniqしてカウントしてあげると、どのルームIDでどのくらい発言があったか確認することができる

```bash
User$ uniq -c ./45b4c87092------------b7b4d256e7f03/zchat.csv  | tail
   3 00028
   1 00007
   8 00028
   4 00002
   8 00007
   2 00002
   1 00007
   2 00002
  36 00007
  25 00002
```

