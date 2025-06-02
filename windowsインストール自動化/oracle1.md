Windows Server 2022でOracle Database 19c (64bit)とOracle Instant Client 19 (32bit)を使用し、IISの32bitウェブサイトでBadImageFormatExceptionを回避する設定について説明します。

## 問題の原因

BadImageFormatExceptionは、64bit環境で32bitのOracle.DataAccess.dllを使用する際に、アーキテクチャの不整合で発生します。

## 解決方法

### 1. Oracle Instant Client 32bitの設定

**インストール場所：**
```
C:\oracle\instantclient_19_x86
```

**環境変数の設定：**
- `ORACLE_HOME`: `C:\oracle\instantclient_19_x86`
- `PATH`に追加: `C:\oracle\instantclient_19_x86`
- `TNS_ADMIN`: `C:\oracle\instantclient_19_x86` (tnsnames.oraがある場合)

### 2. IISアプリケーションプールの設定

**重要な設定項目：**
- **「32 ビット アプリケーションの有効化」**: `True`に設定
- **.NET CLR バージョン**: アプリケーションに応じて設定
- **プロセス モデル**: 必要に応じて調整

### 3. web.configの設定

```xml
<configuration>
  <system.webServer>
    <!-- 32bit強制実行 -->
    <validation validateIntegratedModeConfiguration="false" />
  </system.webServer>
  
  <runtime>
    <!-- .NET Framework使用時 -->
    <loadFromRemoteSources enabled="true" />
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="Oracle.DataAccess" 
                          publicKeyToken="89b483f429c47342" 
                          culture="neutral" />
        <bindingRedirect oldVersion="0.0.0.0-4.122.19.1" 
                         newVersion="4.122.19.1" />
      </dependentAssembly>
    </assemblyBinding>
  </runtime>
</configuration>
```

### 4. アプリケーション側の考慮事項

**参照設定：**
- 32bit版のOracle.DataAccess.dllを参照
- プロジェクトのプラットフォームターゲットを「x86」に設定

**接続文字列例：**
```csharp
string connectionString = "Data Source=localhost:1521/XE;User Id=hr;Password=password;";
```

### 5. 検証手順

1. IISマネージャーでアプリケーションプールの設定確認
2. 環境変数の確認
3. Oracle接続テスト
4. アプリケーションの動作確認

### 6. トラブルシューティング

**よくある問題と解決策：**
- DLLが見つからない場合：PATHの設定確認
- 依然としてBadImageFormatException：アプリケーションプールの32bit設定確認
- Oracle接続エラー：TNS設定とOracle Databaseの稼働状況確認

この設定により、64bit Windows Server上で32bit Oracle Clientを使用するIISアプリケーションが正常に動作するはずです。