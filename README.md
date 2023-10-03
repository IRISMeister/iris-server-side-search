
# VSCODEサーバサイド編集のサーチ機能を有効化する

> こちらの内容は、今後のリリースにより変わる(不要になる)可能性があります。

バージョン2023.2以降で、IRISスタジオが[非推奨](https://jp.community.intersystems.com/node/541666)となったこともあり、VSCODE拡張機能を評価される方も今後増えるかと思います。

既存のCache'資産をお持ちで、ソース管理をソースコントロールフックで実施されている方などにおかれましては、その際にサーバサイド編集を選択される方もおられるかと思います。

VSCODE拡張には、Cache'/IRISスタジオの「ファイルから検索」と同じ要領でサーチを行いたいというご要望に応えるための機能が備わっています。その[導入方法](https://github.com/intersystems-community/vscode-objectscript#enable-proposed-apis)が、VSCODEの未公開APIを使用している関係で、ひと手間かかるものとなっているため、解説します。

> 方法は複数ありますが、手順を簡素化するべく、なるべくGUIを使わない方法をご紹介しています。

# 導入方法

サーバサイドのサーチ機能は、VSCODEの["Proposed API"](https://code.visualstudio.com/api/advanced-topics/using-proposed-api)であるTextSearchProvider,FileSearchProvideを使用しています。
いずれこれらのAPIが安定化してStableとしてリリースされるまでの措置として、これらを使用している拡張機能はマーケットプレイスからの導入(クリックするだけの簡単インストール)が制限されています。

> サーバサイドのサーチ機能を使用する目的でVisual Studio Code Insidersリリースを使用する必要はないようです。

それまでは、マニュアルで導入する必要があります。以降、その導入手順をご紹介します。

## InterSystems ObjectScriptのベータ版を導入

使用中のバージョンの「次のバージョン」のbeta.1をインストールします。

> VSCODE拡張は、日々更新・改良されていますので、常に最新バージョンを保つことをお勧めします。

使用中のバージョンは、下記で確認できます。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/version.png?raw=true)

この例では2.10.3ですので、[2.10.4-beta.1](https://github.com/intersystems-community/vscode-objectscript/releases/download/v2.10.4-beta.1/vscode-objectscript-2.10.4-beta.1.vsix
)をダウンロードし、ダウンロードされたファイル(拡張子vsix)を、Extentionsにドラッグ＆ドロップします。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/dd.png?raw=true)

再起動を促されますので、vscodeを再起動します。

## Proposed APIを有効化

Command Paletteで"Preferences: Configure Runtime Arguments"を選択し、下記を追加します。
```
	"enable-proposed-api": ["intersystems-community.vscode-objectscript"]
```

実体はC:\Users\Windowsユーザ名\.vscode\argv.jsonにありますので、直接編集しても良いです。

```
// This configuration file allows you to pass permanent command line arguments to VS Code.
// Only a subset of arguments is currently supported to reduce the likelihood of breaking
// the installation.
//
// PLEASE DO NOT CHANGE WITHOUT UNDERSTANDING THE IMPACT
//
// NOTE: Changing this file requires a restart of VS Code.
{
	// Use software rendering instead of hardware accelerated rendering.
	// This can help in cases where you see rendering issues in VS Code.
	// "disable-hardware-acceleration": true,

	// Allows to disable crash reporting.
	// Should restart the app if the value is changed.
	"enable-crash-reporter": true,

	// Unique id used for correlating crash reports sent from this instance.
	// Do not edit this value.
	"crash-reporter-id": "xxxxxxxxxxxxx",

	"enable-proposed-api": ["intersystems-community.vscode-objectscript"]
}
```

有効化されると、OUTPUT(ObjectScript)に下記のようなメッセージが表示されるようになります。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/output.png?raw=true)

## VSCODEのワークスペースを作成・保存

VSCODEのワークスペースを作成・保存して、アクセスしたいIRIS環境を追加(感覚的にはマウントする感じです)します。

この作業はUIで行っても良いのですが、ここではワークスペースの設定ファイル(.code-workspace.code-workspace)を直接編集する方法をご紹介します。ワークスペースの設定ファイルをVSCODE自身で編集するには、いったん、フォルダとして開き直します。

編集後の.code-workspace.code-workspaceは下記のようになります。
```
{
	"folders": [
		{
			"name": "local:scdemo",
			"uri": "isfs://local:scdemo/",
		},
		{
			"name": "local:samples",
			"uri": "isfs://local:samples/"
		},
		{
			"path": "."
		}
	],
	"settings": {
		"objectscript.showExplorer": false,
		"intersystems.servers": {
			"local": {
				"webServer": {
					"scheme": "http",
					"host": "localhost",
					"port": 52773
				},
				"username": "SuperUser"
			}
		}
	}
}
```

上記の意味は  
- local:scdemoという名前(name)で,アクセス先サーバ isfs://local/ のネームスペースscdemoを、ワークスペースにマウントする
- local:samplesという名前(name)で,アクセス先サーバ isfs://local/ のネームスペースsamplesを、ワークスペースにマウントする
- フォルダの"."、つまりルートフォルダをワークスペースにマウントする（これはローカルにファイルがなければ不要です）
- ただしサーバlocalの実体は、http://localhost:52773/ 、IRISユーザ名SuperUserを使用する

となります。IRISサーバのホスト名やポート、ユーザ名は、環境に合わせて変更してください。

> nameは、検索対象のフォルダ名の一部として使用されますので、あまり長くすると、サーチ結果を読みにくくなります。

これでサーバサイド編集やサーバサイドサーチを使用する準備が出来ました。ワークスペースとして開き直してください。

ご参考までに、UIで行う方法は下記になります。

[接続するIRISサーバの定義](https://docs.intersystems.com/components/csp/docbook/DocBook.UI.Page.cls?KEY=GVSCO_config#GVSCO_config_addserver)と、ネームスペース指定して、仮想的なファイルシステムであるisfsを[ワークスペースに追加](https://docs.intersystems.com/components/csp/docbook/DocBook.UI.Page.cls?KEY=GVSCO_ssworkflow#GVSCO_ssworkflow_config)します。

## 使用

ここまでの作業を行ったフォルダを[GitHub](https://github.com/IRISMeister/iris-server-side-search)で公開してありますので、ご利用ください。git cloneした後に、フォルダ内の.code-workspace.code-workspaceをワークスペースとして開いて使用します。

うまく設定が出来ていれば、Exploreに下記のようなフォルダが表示されます。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/folder.png?raw=true)

この時点で、各フォルダをクリックすれば、IRISサーバ内の要素がツリー表示され、編集、保存(コンパイル)可能になっているはずです。

>初めてアクセスする場合はパスワードを聞いてきます。以降、The extenstion 'InterSystems Language Server' wants to sign in using InterSystems Server Credentials.と表示されたら、Allowを押して使用するCredential(SupseUser on localなど)を選択してください。この辺りは、通常のサーバサイド編集時の操作と同じです。

肝心のサーチ機能ですが、Control+シフト+Fを押してファイルサーチ機能を開いてください。普通にvscodeでファイルサーチを行う要領で、サーチワードを入力すると、ヒットした対象が列挙されます。大量にヒットした場合、(画像のように)Collapse Allアイコンで全体を折りたたんで、まずはヒット件数表示すると良いかもしれません。 

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/hits.png?raw=true)

files to includeにフォルダ指定するのと同じ要領で、フィルタ設定すると、サーチ対象もフィルタされます。

EnsLibパッケージを対象にする場合。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/filter.png?raw=true)

ワイルドカードを含む場合。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/filter-pkg.png?raw=true)

MACのみを対象にする場合。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/filter-mac.png?raw=true)

名前が分かっている場合は、Quick Open機能を使用できます。フォルダツリーを使って、大量に存在するクラスやルーチンの中から、編集対象をピックアップするのが大変な時に役立ちます。

![](https://github.com/IRISMeister/iris-server-side-search/blob/main/images/quickopen.png?raw=true)
