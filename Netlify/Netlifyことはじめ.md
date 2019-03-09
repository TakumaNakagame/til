# Netlify ことはじめ

## やったこと
- NetlifyにWebサイトをホスティングした
    - GridSomeでポートフォリオサイトを構築したい
    - [@mottox2](https://twitter.com/mottox2)さんにNetlifyを教えてもらって触ってみた
- NetlifyにGridsomeのホスティング
    - Netlifyのアカウント作成
    - GitHubアカウントと連携
    - GridSomeのリポジトリから、Netlifyへデプロイ
        - [gridsome/gridsome-starter-markdown-blog: Markdown blog starter for Gridsome](https://github.com/gridsome/gridsome-starter-markdown-blog)
    - 自動的にWebサイトがGenerateされホスティング。　
        - [https://compassionate-haibt-9a0034.netlify.com/](https://compassionate-haibt-9a0034.netlify.com/)
- カスタムドメインの設定
    - 参考： [【Netlify】カスタムドメインを設定する - Qiita](https://qiita.com/NaokiIshimura/items/64e060ccc244e38d0c15)
    - ドメインを入力
    - CNAMEの登録(サブドメインの場合)が求められるのでDNSに登録
    - 5分ほど待って完了
        - Netlify側でCheck Doneとなっていても、ローカルがキャッシュしている可能性大
        - `dig CNAME v2.www.lab-dev.net` で↑のアドレスが出てくればOK
        - [https://v2.www.lab-dev.net](https//v2.www.lab-dev.net)
- その他
    - Branchごとにサブドメインを分ける構築が可能らしい！
    - [Custom Domains | Netlify](https://www.netlify.com/docs/custom-domains/#branch-subdomains)