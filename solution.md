# IEEE-CIS Fraud Detectionまとめ

## 前書き

こちらのコンペ、私はソロで参加しましたが、正直自分の力でメダルを取れたとは思っていません。
色々なカーネル、Discussionを読み、それらを自分の環境で試し、こういう風になるんだみたいな感じの
ノリでやっていたので、メダルこそ取れはしましたがほぼほぼ、他力本願です。
ですが今回のコンペで少しは学べた事もありますので、それらをまとめて行こうと思います。

ちなみに今回のコンペがkaggleの真剣勝負初参戦です。

## コンペについて

クレジットカードの不正利用を見つけるコンペティションです。

[EEE-CIS Fraud Detection](https://www.kaggle.com/c/ieee-fraud-detection)

データはテーブルデータで与えられており、各取引の情報がレコードとして与えられています。
不正であったかどうかが、**isFraud**カラムに与えられており、testデータのisFraudが1か0を
予測するコンペとなっちます。


isFraudが0のデータが57万件、それに対し1のデータが2万件の不均衡データです。
他にも欠損値が多いデータでもあります。

## 不正検知について

不正検知は、過去の不正利用のパターンと類似した決済が行われた場合に、起きる事が多いそうです。

不正利用のパターンには以下のようなものがあります。

> 前に不正利用があった時と同じ、もしくは近しい方法で利用があった場合

>同じ人が同じ店で、集中して何度もクレジットカードを利用している場合

>ほぼ同時刻に複数の店で、同じクレジットカードが利用された場合

>同じ人が、離れた場所で短い時間にもかかわらず買物した場合

[あなたを守るカード不正利用検知システム](https://www.smbc-card.com/mem/hitotoki/learn/fraud.jsp)


## 使用モデル

僕はカーネルやDiscussionを見ている中で、大半の人がLightGBMを使用しているのが分かったので、LGBMを使用しました。

## メモリ削減

僕自身、何も考えずにpd.read_csvをして、そのままデータを可視化させたりしたら、メモリエラーでブラウザが落ちたり、カーネルが死ぬこともあったので、下記ののように、DataFrameの型を最適化（int64->int32）しました。

ほぼカーネルDiscussion参照です。

```
def reduce_mem_usage(df):
    """ iterate through all the columns of a dataframe and modify the data type
        to reduce memory usage.        
    """
    start_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage of dataframe is {:.2f} MB'.format(start_mem))
    
    for col in df.columns:
        col_type = df[col].dtype
        
        if col_type != object:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
        else:
            df[col] = df[col].astype('category')

    end_mem = df.memory_usage().sum() / 1024**2
    print('Memory usage after optimization is: {:.2f} MB'.format(end_mem))
    print('Decreased by {:.1f}%'.format(100 * (start_mem - end_mem) / start_mem))
    
    return df
```

## EDA

EDAは下記のカーネルを参照して、データの理解に勤めました。

[EDA and models](https://www.kaggle.com/artgor/eda-and-models)

上記のカーネルのおかげで、あらかたデータの概要がわk流事ができました。
正直、最初で出たを見たときに、僕はcardという絡むが何を表しているのかよくわかりませんでしたが、Discussionに
データの詳細がわかるものと、こちらのカーネルで理解する事ができました。カードは簡単に表すとカテゴリらしいです。

## パラメータとモデル

この辺は下記のカーネルを参考にして行いました。

[IEEE - FE for Local test](https://www.kaggle.com/kyakovlev/ieee-fe-for-local-test)

## 学んだこと

- カーネルとdiscusiionを見ることの重要性

これに関しては僕は絶対するべきだと思っています。実際あんまり理解していない僕でも、カーネルとDiscussionを見てそれを
実行することによってメダルを獲得する事ができました。
重要な情報はだいたい、カーネルとDiscussionに世界中のデータ分析大好き人間が参加していうrのでそれを見るだけでも
自分の知識量増加につながると思いました。またカーネルでがっつりスコアが高くでる手法を公開しているパターンもあります。(自分のメダル獲得はほぼほぼ、それの恩恵)

また僕は自分なりの特徴量だったりをそこまで作れませんでしたが、他者が作った特徴量や情報に重要度をつけて、実際に試してみるという進め方は
めちゃくちゃ弁キュ王になると思うので、次テーブルデータコンペがきたらやろうと思っています。

- 特徴量を追加し分析した後、精度が向上しなかった場合、なぜ効かなかったのかを考えること

これは僕は、他者のカーネルをほとんど参考にしていたのでなんとも言えませんが、少なくともなぜ特徴量が効かなかったのかを
考える必要はめちゃくちゃあって、それこそ僕みたな人間は、他者の優秀なカーネルを参考にしていたので、他者が作った特徴量を
理解することぐらいは、しっかりしなきゃと思って取り組みました。
そのおかげもあってか、制度が上がる特徴量の傾向がなんとなく分かるようになりました。

## 最後に

コンペを１から取り組んで最後まで走り切ったのは今回が初めてで、しかもメダルまで獲得できるとは思っていませんでした。
しかし完全に自分の実力ではないので嬉しいですが、心の中では全然喜べていません。もっと独自の取り組み方を行い、
本当の自分の結果を残したいと思いました。

## 結果


![スクリーンショット 2019-10-05 12 00 59](https://user-images.githubusercontent.com/38784824/66250960-c1861500-e784-11e9-9a81-58a7833bb2ec.png)