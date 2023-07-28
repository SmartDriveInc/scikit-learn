# Wheel のビルド方法

## macOS 上で macOS 用の wheel をビルドする

まず Python の仮想環境を作成し、それを有効化します。

```shell
$ python -m venv .venv
$ source .venv/bin/activate
```

そして以下のコマンドを実行して、 macOS 用の wheel をビルドします。

```shell
$ pip install build
$ python -m build --wheel
```

これにより macOS 用の native extension がビルドされ、以下のような wheel が生成されます。

```
dist/scikit_learn-0.24.2-cp3XX-cp3XX-macosx_XX_0_(x86_64|arm64).whl
```

## macOS 上で Linux 用の wheel をビルドする

Linux 用の wheel をビルドするのには Docker を利用します。
使用する Docker イメージ (`python:3.XX-<DEBION_VERSION>`) にはビルドに使用したい Python バージョンのものを指定してください。
なお Apple Silicon Mac 上で x86_64 用の wheel をビルドする場合には時間がかかります。

```shell
$ docker container run \
    --rm \
    --platform=linux/amd64 \
    --mount=type=bind,source=${PWD},target=/src \
    --workdir=/src \
    python:3.XX-<DEBION_VERSION> \
    bash -c "pip install build && python -m build --wheel"
```

これにより Linux 用の native extension がビルドされ、以下のような wheel が生成されます。

```
dist/scikit_learn-0.24.2-cp3XX-cp3XX-linux_x86_64.whl
```


# ビルドした wheel を Google Artifact Registry にアップロードする方法

まず最初に [gcloud CLI](https://cloud.google.com/sdk/docs/install-sdk) で以下のコマンドを実行し、ブラウザで Google 認証をおこないます。

```shell
$ gcloud auth application-default login
```

これにより Google Artifact Registry (GAR) にアクセスする際の認証に使用される application default credentials (ADC) が設定されます。

続いて Python の仮想環境を作成し、それを有効化します。
ビルドしたときに作成した仮想環境があるなら、それを再利用しても OK です。

```shell
$ python -m venv .venv
$ source .venv/bin/activate
```

そうしたら Python パッケージをレジストリにアップロードするためのツールである [Twine](https://twine.readthedocs.io/en/stable/) と、 GAR にアクセスするために必要となる GAR 用の keyring バックエンドである [keyrings.google-artifactregistry-auth](https://github.com/GoogleCloudPlatform/artifact-registry-python-tools) をインストールします。

```shell
$ pip install twine keyrings.google-artifactregistry-auth

# GAR 用 keyring バックエンドが期待通りにインストールされたかどうか確認する
$ keyring --list-backends
```

これで一覧に `keyrings.gauth.GooglePythonAuth` が含まれていれば OK です。

最後に以下のコマンドを実行して、ビルドされた wheel を指定の GAR リポジトリにアップロードします。
このコマンド例では `dist/` ディレクトリ以下のすべての wheel をアップロードする形になっていますが、特定の wheel だけをアップロードしたい場合にはその wheel のパスを指定してください。

```shell
$ python -m twine upload \
    --repository-url https://<LOCATION>-python.pkg.dev/<PROJECT_ID>/<REPOSITORY> \
    dist/*
```

なお同名の wheel がすでにアップロード済みである場合にはアップロードに失敗するため、上書きしたいときには既存の wheel を削除してからアップロードしてください。