vimwiki で Github生活
=====================

* https://qiita.com/hinoshiba/items/b931fcd06c278be5b7ca


# 何をしたいか
* vimが大好き！コンソールから離れたくない！
* vimのプラグインの組み合わせで良さげなメモ環境を作っておきたい！
* できればMarkdown！そしてコンソールが握れないときはgithubで見たい！
 * これの対応でちょっといじったメモが今回の内容
* 今までの利用方法ではいくつか難しい点が出てきた

## 今までの環境は以下のプラグインの組み合わせ
* vimwiki
 * vimでMarkdownが編集できるようになるplugin
 * 見出しの作成やToDoリストのトグルなどもコマンドで実行できる
 * リンクを生成し、ファイルの自動作成が可能。Pathが正しければリンクの移動も可能。
 * https://github.com/vimwiki/vimwiki
* VOoM
 * ファイルの種類に応じてツリーを表示してくれるplugin
 * ファイルごとのツリーを表示してくれる
 * つまり移動しやすいのは１ファイルで作業すること
 * ツリーの表示/非表示の切り替えが可能
 * voom vimwiki で呼び出すことで、wikiのツリー構造を表示してくれる
 * http://vim-voom.github.io/

## 今までの環境における問題点 
* １ファイルで完結するため、gitにアップロードした時に見づらい
* １万行超えてたあたりから、VOoMのツリー表示が遅くなってきた
* VOoMを非表示にした場合、１ファイルのwikiファイルを移動することになるので、移動性が悪い

## やりたくなったこと
* wikiファイルを複数に分けたい
* vimwikiのリンクを用いた移動機能を最大限に生かしたい
* できれば、gitアップロード後もリンクの移動を利用したい

# 新環境（vimwiki + NERDTree + Github）
## 導入プラグインの紹介
* vimwiki
 * 上記参照
* NERDTree
 * vim利用できるファイルエクスプローラplugin
 * ファイルやディレクトリの作成、コピー、移動、削除も可能
 * ツリーの表示/非表示の切り替えが可能
 * https://github.com/scrooloose/nerdtree

## 改善した状況
### 今までの環境における問題点 
* １ファイルで完結するため、gitにアップロードした時に見づらい
 * vimwikiの機能を用い、複数ファイルの移動ができる！
 * 複数ファイルのツリーをNERDTreeで確認できるため、表示ページに記載のないリンクの移動も楽
* １万行超えてたあたりから、VOoMのツリー表示が遅くなってきた
 * １６階層３００ファイルぐらいで利用している環境もあるが、まだそんなに重くない！
* VOoMを非表示にした場合、１ファイルのwikiファイルを移動することになるので、移動性が悪い
 * vimwikiの機能を用い、リンクの移動が可能！

## 導入
* neobundleを利用！なので割愛
 * https://github.com/vimwiki/vimwiki
 * https://github.com/scrooloose/nerdtree
 * 僕のvimrcはこんな感じ（https://github.com/hinoshiba/DotFiles/blob/master/all/.vimrc）
* github
 * 登録して、リポジトリほっておく

#　新環境における問題点と対応
## githubとvimwikiのリンクの違いがあった
* vimwikiで自動生成するリンクは以下
 * [[testfile]]
* vimwikiで自動生成するファイル名は以下
 * testfile.wiki
* githubのリンクの解釈はファイル名をそのまま扱うので差異が生じ、リンクを利用することができない！
 * つまり、[[testfile.wiki]]というリンクである必要がある

## プラグインの編集位置
* なので、ファイル名がそのままリンクになるように手を加える

```diff
 ~/.vim/bundle/vimwiki/autoload/vimwiki/base.vim
  644   if vimwiki#base#mkdir(dir, 1)
+ 645     let fname =  substitute(fname,VimwikiGet('ext').'$',"","g")
  646     execute a:command.' '.fname
- 710     let newline = strpart(line, 0, ebeg).a:sub.strpart(line, ebeg+elen)
+ 710     "let newline = strpart(line, 0, ebeg).a:sub.strpart(line, ebeg+elen)
+ 711     let newline = strpart(line, 0, ebeg).substitute(a:sub,"\]\]",".wiki\]\]","g").strpart(line, ebeg+elen)
```

## 注意点
* vimwikiのリンク自動生成機能で、属性が付いていないテキスト上でEnterキーを入力することで、リンクになる機能が存在する
 * ~~本機能を用いる際、.wikiをつけないテキスト上でEnterキーを入力した場合、リンク先のファイルが.wiki出ないため、プラグインが読み込まれず、詰む。~~
 * 2018/03/25にプラグインの編集位置に追記して改善

