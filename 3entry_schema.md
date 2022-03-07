#### エントリスキーマ -[*5.エントリスキーマとユーザ定義項目*](https://vte.cx/documentation.html#index05)
ATOM項目以外の項目をエントリスキーマに記述することでユーザ独自の項目を定義することができます。実際に業務アプリケーションを作成する場合にはこのユーザ定義項目をいかに定義していくかが鍵になります。
　エントリスキーマにユーザ項目を定義するには、管理画面の「エントリスキーマ管理」で行うか、プロジェクトの/setup/_settings/template.xmlファイルを直接編集し、項目名などを記述することでできます。（管理画面で編集を行った後は、必ずプロジェクトのtemplate.xmlファイルも更新するようにしてください）
　システム運用中でもファイルの更新は可能であり、項目を追加して更新すると直ぐにシステムに反映されます。ただし、追加項目は常に要素の最後尾に追記する必要があります。途中の階層であっても同列の最後であれば項目を追加できます。

エントリスキーマの記述ルール

エントリスキーマには以下の記述ルールが適用されます。ルールを適用できるものは、項目の有無や親子関係、繰り返しといった構造の定義、数値や文字列、日付などの型の指定、必須チェックや最大/最小チェック、正規表現によるバリデーションチェックなどがあり、データ登録更新時にルールに合致しなければエラーとなります。

項目名(型){多重度}!=正規表現 の形式で項目を記述します。
項目名は２文字以上１２８文字以下の英数字および一部の記号(_や$)が使えます。ただし、数字で始まるものやハイフンは使用不可です。
インデントが下がると上記要素の子要素であることを示します。
項目名()の中には型を指定します。型には、string,int,date,long,float,double,boolean,descがあります。
()を省略した場合や前述した型以外が指定された場合はstringになります。型名は先頭は大文字小文字のどちらでも構いません。(case sensitiveではない)
string型に格納できる文字の最大サイズは10MiBです。これを超えるラージオブジェクトについては項目ではなくコンテンツとして扱ってください。
date型は以下の形式のものを受け付けます。
yyyy-MM-dd
yyyy-MM-dd HH
yyyy-MM-dd HH:mm
yyyy-MM-dd HH:mm:ss
yyyy-MM-dd HH:mm:ss.SSS
上記の"-"を"/"にしたもの
上記の" "を"T"にしたもの
yyyyMMdd
yyyyMMddHH
yyyyMMddHHmm
yyyyMMddHHmmss
yyyyMMddHHmmssSSS
上記の各フォーマットについて、末尾にタイムゾーン([ISO 8601] +99:99、+9999、+99)を加えたもの
desc型は降順ソートを行うための項目となります。{降順ソートしたい項目名}_desc(desc)と指定してください。
例えば、prop1_desc(desc)をエントリスキーマに定義するとprop1の値で降順ソートされます。
項目名{} はmapを意味します。括弧の中は要素数の最大値を示し、省略すると1になります。ただし、int型やstring型の場合は要素数の最大値ではなく、int型の場合は最大値、string型の場合は最大の長さになります。
要素数には子要素の最大繰り返し数を指定します。
item(int){3}の場合、itemに設定できる値は3が最大値となります。
item2(string){3}の場合、item2に設定できる文字数が3文字となります。
項目名の最後に!を付けると必須項目となります。
注意：!は型や繰り返しよりも後に記述します。=の直前です。
=に続けて正規表現を指定することでバリデーションを定義できます。指定した正規表現にマッチしないとエラーとなって入力を受け付けません。
正規表現構文については、Regex Patternを参照してください。
正規表現が指定されている項目はデータ登録時にバリデーションチェックを実行します。
$によりXMLの属性やテキストノードを指定することができます。
項目名の先頭が$のものはXMLにシリアライズしたときに属性となります。ただし、属性は同列の他の項目より先に記述する必要があります。
$$textを付けることでテキストノードとみなされ、子要素ではなく自身の要素に値を代入します。
title、subtitle、updatedといったATOM項目については、エントリスキーマに定義しなくても使えます。逆に定義済みであるATOM項目をユーザ定義項目として定義しようとすると重複エラーとなります。
以下はエントリスキーマのサンプルです。/_settings/templateエントリの<content>タグに以下を記述することでエントリスキーマを設定できます。

idx
email
verified_email(Boolean) // Boolean型
name
given_name
family_name
error
 errors{2} 　　　　　　// インデントでerrorの子要素。Mapで多重度は2
  domain
  reason
  message
  locationType
  location
 code(int){1~100}       // 1~100の範囲
 message
subInfo
 favorite
  $attribute       // $で始まる項目はXMLの属性となる（項目の先頭に記述する）
  food!=^.{3}$  　　// !で必須項目を示す。もし{}があればそれよりも後に記述する。 food{}!=xxx など
  music=^.{5}$
 favorite2
  food
   food1
 favorite3
  food
 hobby{}
  $$text           // $$textはXMLのテキストノードになる
上記エントリスキーマを利用するサンプルアプリケーションでは以下のようなリクエストになります。リクエストでは、スキーマに定義されている項目のうち実際に必要なものだけが使用され、また、リクエストの第一階層の項目に値が代入されていないものは出現しない項目となります。
　例えば、このエンドポイントへのリクエストには、emailやfamily_name、subinfoなどの項目だけが存在する一方で、errorなどの項目はレスポンスだけに存在します。
　このように、アプリケーションを設計する際、どのエンドポイントでどの項目を使うかについてグルーピングの定義が必要です。vte.cxでは管理画面の「エンドポイント管理」を利用してこれらを定義することができます。

POST /d/registration

[
            {
                "email": "email1",
                "family_name": "管理者Y",
                "given_name": "X",
                "name": "管理者",
                "subInfo": {
                    "favorite": {
                        "food": "カレー",
                        "music": "ポップス1"
                    }
                },
                "verified_email": false
            }
        ]

レスポンス

[
            {
                "error": {
                    "code": 400,
                    "errors": [
                        {
                            "domain": "vte.cx",
                            "location": "Authorization",
                            "locationType": "header",
                            "message": "invalid header",
                            "reason": "invalidAuthentication"
                        }
                    ],
                    "message": "Syntax Error"
                }
            }
        ]
エントリスキーマとユーザ定義項目