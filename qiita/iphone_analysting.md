嫁の携帯で怪しいものを発見した(のて調べ方手順メモしてみる)
=======================

* https://qiita.com/hinoshiba/items/6c83871e7b521864ba60

# 何がおきたか
* ふと、嫁のIPhoneのバックアップが７日間取得できていないことに気がつく
* どうやら原因は、アンチウイルスソフトがバックアップファイルの中に怪しいものを見つけて、バックアップの途中で削除してしまっている様子
* 結果、IPhoneのバックアップが異常になってしまい、バックアップに失敗しているぽい。

# 調査
## アンチウイルスソフトのログからファイル名を特定
* これは、各種アンチウイルスソフトによると思うので、割愛
* 今回は以下に発見

```
/Users/[my]/Library/Application Support/MobileSync/Backup/[backupname]/Snapshot/b7/b79d2[ファイル名続く]
```

## ファイル名から、アプリケーションの特定
1. DBファイルを開く（DB Brows for SQLiteを使用）
 * `/Users/[my]/Library/Application Support/MobileSync/Backup/[backupname]/Manifest.db`
2. FilesテーブルのfileIDで検索
 * 調査で確認した、”b79d2[ファイル名続く]”の部分
3. 検索結果から、アプリケーションを特定

## 今回の結果
* SMSアプリケーションの添付保存先に、対象を確認
 * bitcoinのマイニングプログラムのダウンローダVBScriptがzipで添付メールとして送られていたっぽい。
 * そして、Windowsでしか動作しないコードだった。
* 影響がなかったので、安心安心。

