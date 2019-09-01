# ml-competition-IEEE-CIS_Fraud_Detection

- kaggle ml competition **IEEE-CIS_Fraud_Detection** management repository

## Structures

```
├───.gitignore                     # 追跡対象としないファイルを指定するための設定ファイル
├───data                           # データ一覧
│   ├───external　　　　　　　　　　  ＃ submission以外のcsvを置くDirectory
│   ├───features                   # 特徴量を保存するdirectory
│   ├───input                      # Kaggleから落としたcsv全般を保存しておくdirectory
│   │   └───.gitignore
│   ├───model                      # モデルを保存するdirectory
│   └───submit                     # Submission用csvを保存するdirectory
│───features                       # 特徴量を生成するコンペ特有なコード(.py)を置くdirectory
│   └───convert_to_feature.py 
│───importance                     # 特徴量のimportanceをcsvとして保存しておくdirectory
│───notebooks
│   ├── deprecated                 # 使う価値のないipynbを追いやるdirectory      
│   └───eda                        # EDAを行ったipynbを置くdirectory
│───tmp                            # どうしても一時的になにかを置きたいときに使うdirectory(gitで共有はしない)
└───README.md                      # README.mdです。
```

## Rule

- ブランチ名は基本的に、**やるべきこと-名前** の様にする。

```
# 例: 特徴量を新しく作成するなら以下の様にする。
make_feature-tsuruse
```

- 基本的にmasterで作業しない。

- マージする時は、一声かける。