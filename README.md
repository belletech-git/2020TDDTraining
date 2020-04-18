# 3. モックを使ってServiceのテストを書く
やっとServiceクラスにテストが書ける状態になったので、早速書いていきましょう

## テストの書き方
Railsが提供する `ActiveSupport::TestCase` クラスを継承した `CreatePhotobookServiceTest` クラスを作ります

そこでは、以下のような形式でテストが書けます

```rb
def sum(x, y)
  x + y
end

class SumTest < ActiveSupport::TestCase
  test '1+1=2であること' do
    assert_equal(2, sum(1, 1))
  end  
end
```

基本的には、assertメソッドの左側に「期待する値」, 右側に「テストしたい処理」を書きます

## モックの作り方
基本的には先の問題のREADMEで書いたようなものを作ります

大事なのが、「同じInterfaceを実装すること」です

例えば、Albumのモックを作るならば以下のような形になりそうです

```rb
class MockAlbum
  attr_accessor :children

  def initialize(children:)
    @children = children
  end

  def find_by(id:)
    MockAlbum.new(children: @children)
  end
end
```

この要領でモックをModelに対してそれぞれ作成し、イニシャライザ引数でServiceに渡していくわけです

## テスト項目
テストは以下のコマンドで実行できます

```
$ docker-compose run web rails test test/services/create_photobook_service_test.rb
```

### 1. 正しいタイトル, サブタイトルのフォトブックが作成されること
例えば、以下のような入力があった場合

```
子供:
  1: { name: 太郎, birthday: 2019/01/01 }
  2: { name: 次郎, birthday: 2020/01/01 }
カバー写真の撮影日: 2020/04/01
```

出力は以下のようになります
```
title: 2020年4月のアルバム
subtitle: 太郎1歳3ヶ月・次郎0歳3ヶ月
```

### 2. フォーマットされたサブタイトルが文字数制限を超えていた場合、デフォルトのサブタイトルに指し替わっていること
- デフォルトのサブタイトルは `Family Album` です
- サブタイトルの長さの上限は `20` 文字です

### 3. パラメータで渡されるタイトルとサブタイトルがnilでなければ、そのタイトルとサブタイトルでフォトブックを作成すること
引数でtitleとsubtitleに任意の文字列を指定すると、返り値のPhotobookのtitleとsubtitleがその値と等しくなっていることを確認しましょう

### 4. フォトブックのカバー写真撮影日より子どもの誕生日が後の場合、サブタイトルに年齢を表示しないこと
例えば、以下のような入力があった場合

```
子供:
  1: { name: 太郎, birthday: 2019/01/01 }
  2: { name: 次郎, birthday: 2020/01/01 }
カバー写真の撮影日: 2018/04/01
```

subtitleは以下のようになります
```
subtitle: 太郎・次郎
```

### 5. 該当するアルバムが存在しない場合、 `AlbumNotFoundError` をraiseすること
以下のassertメソッドでraiseはテストできます

```rb
assert_raise(CreatePhotobookService::AlbumNotFoundError) do
  service.call(...)
end
```

アルバムがnilになるようモックを作ります

### 6. 引数のタイトルの長さが上限を超えた場合、 `TitleTooLongError` をraiseすること
タイトルの長さ上限20を超える文字列をtitle引数に渡してみましょう

### 7. 引数のサブタイトルの長さが上限を超えた場合、 `SubtitleTooLongError` をraiseすること
サブタイトルの長さ上限20を超える文字列をsubtitle引数に渡してみましょう

### 8. 引数のフォトブックのコメントの長さが上限を超えた場合、 `CommentTooLongError` をraiseすること
コメントの長さ上限200を超える文字列をcomment引数に渡してみましょう

## TODO
- [ ] テスト項目1を実装
- [ ] テスト項目2を実装
- [ ] テスト項目3を実装
- [ ] テスト項目4を実装
- [ ] テスト項目5を実装
- [ ] テスト項目6を実装
- [ ] テスト項目7を実装
- [ ] テスト項目8を実装

## 答え
https://github.com/mixi-inc/2020TDDTraining/compare/question-3...answer-3
