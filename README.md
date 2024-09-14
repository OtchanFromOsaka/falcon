# Project Falcon

### Identity Center 設定

CloudShell を起動して以下のコマンドを実行する

```
read -p "Enter email address: " user_email # hit enter
```

```
| Enter email address: <your-email-address>
```

続けて以下のコマンドを実行する ※ `xxxxxxxxxx` と `Xxxxxxxxxx` をコードネームに置き換える

```
response=$(aws sso-admin list-instances)
ssoId=$(echo $response | jq '.Instances[0].IdentityStoreId' -r)
ssoArn=$(echo $response | jq '.Instances[0].InstanceArn' -r)
email_json=$(jq -n --arg email "$user_email" '{"Type":"Work","Value":$email}')
response=$(aws identitystore create-user --identity-store-id $ssoId --user-name xxxxxxxxxx-admin --display-name 'XxxxxxxxxxAdmin' --name Formatted=string,FamilyName=Admin,GivenName=xxxxxxxxxx --emails "$email_json")
userId=$(echo $response | jq '.UserId' -r)
response=$(aws sso-admin create-permission-set --name xxxxxxxxxx-policy --instance-arn=$ssoArn --session-duration PT12H)
permissionSetArn=$(echo $response | jq '.PermissionSet.PermissionSetArn' -r)
aws sso-admin attach-managed-policy-to-permission-set --instance-arn $ssoArn --permission-set-arn $permissionSetArn --managed-policy-arn arn:aws:iam::aws:policy/service-role/AmplifyBackendDeployFullAccess
accountId=$(aws sts get-caller-identity | jq '.Account' -r)
aws sso-admin create-account-assignment --instance-arn $ssoArn --target-id $accountId --target-type AWS_ACCOUNT --permission-set-arn $permissionSetArn --principal-type USER --principal-id $userId
# Hit enter
```

続けて以下のコマンドを実行する ※ `xxxxxxxxxx` をコードネームに置き換える

```
printf "\n\nStart session url: https://$ssoId.awsapps.com/start\nRegion: $AWS_REGION\nUsername: xxxxxxxxxx-admin\n\n"
```

```
| Start session url: https://d-XXXXXXXXXX.awsapps.com/start
| Region: ap-northeast-1
| Username: xxxxxxxxxx-admin
```

ユーザー -> xxxxxxxxxx-admin -> パスワードをリセット でパスワードを設定する

### ローカル環境構築

Dockerコンテナに入って以下のコマンドを実行する

```
aws configure sso
```

コンソールにURLが表示されるのでブラウザで開いて `XXXX-XXXX` を貼り付ける

```
| SSO session name (Recommended): xxxxxxxxxx-admin
| SSO start URL: https://**********.awsapps.com/start
| SSO region: ap-northeast-1
| SSO registration scopes [sso:account:access]: <leave blank>
|
| https://device.sso.us-east-2.amazonaws.com/
|
| Then enter the code:
|
| XXXX-XXXX
```

`xxxxxxxxxx-admin` でログインして最後まで進むとコンソールに以下が表示されるのでそれぞれ入力する

```
| The only AWS account available to you is: 000000000000
| Using the account ID 000000000000
| The only role available to you is: xxxxxxxxxx-policy
| Using the role name "xxxxxxxxxx-policy"
| CLI default client Region [None]: ap-northeast-1
| CLI default output format [None]: <leave blank>
| CLI profile name [xxxxxxxxxx-policy-000000000000]: default
|
| To use this profile, specify the profile name using --profile, as shown:
|
| aws s3 ls --profile default
```

### Amplify 設定

リポジトリを連携して一度ビルドしてから以下コマンドを実行する ※それぞれ数分かかる

```
npx ampx sandbox
```
