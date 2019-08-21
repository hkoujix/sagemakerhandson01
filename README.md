# SageMaker Sandbox handson

@(Handson)


## Step1 CloudFormationによる環境作成
事前に配布した２つのYAMLファイルのCloudFormation テンプレートを実行します、
* sandbox-landing.yaml
* sandbox-sandbox.yaml

![Alt text](img/1565601124147.png)


管理コンソールからCloudFormationのサービスを選択
![Alt text](img/1565178561912.png)


##### Landingのスタック作成 
![Alt text](img/1565179156581.png)
1.  「新しいスタックの作成」を選択

![Alt スタックの作成](img/1565177361873.png)
2. 「Amazon S3 テンプレート URL の指定」で
 `https://s3-ap-northeast-1.amazonaws.com/kh-handsondata/cfn/sandbox-landing.yaml` を指定し、「次へ」
![Alt text](img/1566385617293.png)

3. スタックの名前に  ”WS1” を入れて「次へ」 

![Alt text](img/1565177868308.png)
4.   デフォルトのまま「次へ」

![Alt text](img/1565177842075.png)
5.  **「AWS CloudFormation によってカスタム名のついた IAM リソースが作成される場合があることを承認します。」** のチェックをオンにして、「作成」

![Alt text](img/1565179414457.png)

##### Sandboxのスタック作成 
![Alt text](img/1566386466234.png)

6. 「新しいスタックの作成」を選択し、「「Amazon S3 テンプレート URL の指定」で
 `https://s3-ap-northeast-1.amazonaws.com/kh-handsondata/cfn/sandbox-sandbox.yaml` を指定する
![Alt text](img/1566386028310.png)

7.  スタック名を 「WS2」にして同様に作成する
![Alt text](img/1566386515390.png)

8. **「AWS CloudFormation によってカスタム名のついた IAM リソースが作成される場合があることを承認します。」** にチェックをオンにして、「作成」
![Alt text](img/1566386573923.png)


9.  スタックが CREATE_COMPLETE になれば完成
![Alt text](img/1565186771844.png)
 
##### 確認
* WS1/WS2 の2つのVPCが作成されている
![Alt text](img/1565186827662.png)


* WS1/WS2 の４つのサブネットが作成されている
![Alt text](img/1565186850900.png)

* endpointインターフェースが作成されている
![Alt text](img/1565186877300.png)

 
* ２つの Notebook インスタンスが作成されている
![Alt text](img/1565186904095.png)



## Step2 カスタムコンテナ作成
Amazon SageMaker > ノートブックインスタンス
* WS1-LandingNotebook インスタンスで　Open Jupyter をクリック
![Alt text](img/1565187027995.png)

* /khlab-handson/scikit_custom/ create_docker.ipynb をオープン
![Alt text](img/1565593540338.png)

* notebookに従って、一行ずつに実行
Shift + Enter でカーソル行を実行します。
![Alt text](img/1565593338297.png)

* docker push が成功すればOK
![Alt text](img/1565593801485.png)

* Notebookを最後まで実行し、S3に関連ファイルをアップロードする
![Alt text](img/1565594252890.png)

このS3のパスをつぎのステップで使います。　パスをメモ帳などにコピー＆ペストしておいてください。

#### 確認
* ECRにイメージが登録されていることを確かめる
![Alt text](img/1565593888093.png)

ECR > リポジトリ
![Alt text](img/1565594155066.png)

![Alt text](img/1565594301510.png)


## Step3 Sandbox環境でモデル学習と推論デプロイ
Amazon SageMaker > ノートブックインスタンス
WS2-SandboxNotebook インスタンスで　Open Jupyter をクリック
![Alt text](img/1565187009296.png)

### 準備作業
* Jupyter からTerminalをオープン

 ![Alt text](img/1566388049970.png)


* S3からnotebook関連ファイルをコピー
Files > New > Terminal 
Terminal で次のコマンドを実行します。
以下の**S3のパス**は、Step２でアップロードしたパスに置き換えてください。
```
cd SageMaker
aws s3 cp --recursive  s3://sagemaker-ap-northeast-1-<アカウントID>/LAB-handson sciket-custom
```



### endpoints.jsonを修正
/home/ec2-user/anaconda3/envs/python3/lib/python3.6/site-packages/botocore/data/endpoints.json
 の hostnameにVPC endpointのホスト名を追記していく

* VPC > エンドポイント
WS2でフィルターをかけて、WS2-SandboxVPCのエンドポイントのみを表示する
![Alt text](img/1566389618917.png)


* エンドポイントを選択して詳細を表示する
![Alt text](img/1566389705174.png)

* エンドポイントの詳細にあるDNS名を１つ選んで、endpoints.jsonの該当サービスのhostnameとして登録していく
![Alt text](img/1566389743608.png)

* 登録するエンドポイント
 *  sts
 * logs
 * notebook
 * api.sagemaker
 * runtime.sagemaker
 *  api.ecr


* endpoints.jsonにhostname登録の例
/home/ec2-user/anaconda3/envs/python3/lib/python3.6/site-packages/botocore/data/endpoints.json
エンドポイントのDNS名(vpceで始まる)を　`hostname` として 追加した例です。

```
      "sts" : {
        "defaults" : {
          "credentialScope" : {
            "region" : "ap-northeast-1"
          },
          "hostname" : "vpce-07a04bf233a7bbccb-yxh1tfbr.sts.ap-northeast-1.vpce.amazonaws.com" 
        },
        "endpoints" : {
          "ap-east-1" : {
            "credentialScope" : {
              "region" : "ap-east-1"
            },
            "hostname" : "sts.ap-east-1.amazonaws.com"
          },
          "ap-northeast-1" : {
            "credentialScope" : {
              "region" : "ap-northeast-1"
            },
            "hostname" : "vpce-07a04bf233a7bbccb-yxh1tfbr.sts.ap-northeast-1.vpce.amazonaws.com"
          },
```
```
      "api.sagemaker" : {
        "endpoints" : {
          "ap-northeast-1" : {
            "hostname" : "vpce-0a8e80e89e089bef4-0oxjfya5.api.sagemaker.ap-northeast-1.vpce.amazonaws.com"
          },
```

```
      "logs" : {
        "endpoints" : {
          "ap-east-1" : { },
          "ap-northeast-1" : {
             "hostname" : "vpce-07b1d8b6b579c9244-9xycdsvm.logs.ap-northeast-1.vpce.amazonaws.com"
          },          
```

```
      "runtime.sagemaker" : {
        "endpoints" : {
          "ap-northeast-1" : {
            "hostname" : "vpce-0b91622fd0b7b989a-xi93pg1w.runtime.sagemaker.ap-northeast-1.vpce.amazonaws.com"
          }
        }
      },
```
```
      "api.ecr" : {
        "endpoints" : {
          "ap-east-1" : {
            "credentialScope" : {
              "region" : "ap-east-1"
            },
            "hostname" : "api.ecr.ap-east-1.amazonaws.com"
          },
          "ap-northeast-1" : {
            "credentialScope" : {
              "region" : "ap-northeast-1"
            },
            "hostname" : "vpce-0a1d76bfb01161444-m5pyrmpt.api.ecr.ap-northeast-1.vpce.amazonaws.com "
          },
```

### モデル学習の実行
* / scikit-custom / scikit-learn.ipynb を開く
![Alt text](img/1566390197703.png)


* notebookに従って、一行ずつに実行
Shift + Enter でカーソル行を実行します。
![Alt text](img/1565600796526.png)


