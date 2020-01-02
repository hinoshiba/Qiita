Redmine トップページをプロジェクトwiki　＆　メール通知
========================

* https://qiita.com/hinoshiba/items/9c69ce9bd53cad6c1b49

#何が起きたか。
* Redmineのトップページをかっこよく wikiを参照するようにした！
* あれ、でもメールが送れない・・・。テストメールすると以下のエラーが表示される。
 * ```No route matches {:action=>"index", :controller=>"welcome"}```

#対応
調べた結果、viaオプションを使いましょうとのこと。

```diff:/var/lib/redmine/config/routes.rb
  21   root :to => 'wiki#show', :project_id => 'hogehogeprj', :as => 'home'
+ 22   match '/', :to => 'welcome#index', via: :all
```

#参考
* http://www.redmine.org/issues/11284

