# setup weirdness

Did the usual stuff. Followed this tutorial:

https://developer.1password.com/docs/connect/get-started/?method=1password-cli#get-help

Did the argocd helm trick where you go to the helm url, add index.yaml, and pick out the version to make argo happy. That works. Also did the trick to combine the helm chart and the local dir in my repo. That works.

Following the 1password setup docs, it works your way through the process of creating a 1password-credentials.json file and also a connect token.

The 1password-credentials.json file is how you configure the connect server. the connect token is how you authenticate to the connect server and pull secrets from it (bearer auth).

All of this goes super smoothly right, you need to upload the credentials file so you run

```sh
kubectl create secret generic op-credentials --namespace=1password-connect --from-file=1password-credentials.json
```

Then your server starts up fine, says it can find the credentials, but then you get a weird error message when you make the first request to the server. It tells you to go look at the server logs.

server logs:
```
{"log_message":"(E) Server: (unable to get credentials and initialize API, retrying in 30s), Wrapped: (failed to FindCredentialsUniqueKey), Wrapped: (failed to loadCredentialsFile), Wrapped: (LoadLocalAuthV2 failed to credentialsDataFromBase64), illegal base64 data at input byte 0","timestamp":"2022-07-05T19:55:41.177187902Z","level":1}
```

Weird as heck, I did just upload the credentials file and it should be fine.

Turns out the credentials file needs to be double base64 encoded, seemingly due to a bug (a feature?) in helm. When you deploy the app using argocd, you do the normal thing of only base encoding it once.

```sh
mv 1password-credentials.json  1password-credentials.json-raw
base64 -w 0 < 1password-credentials.json-raw > 1password-credentials.json
kubectl delete secret op-credentials --namespace=1password-connect
kubectl create secret generic op-credentials -namespace=1password-connect --from-file=1password-credentials.json
```

And it works now.

[Thank you "Former Member"](https://www.1password.community/discussions/developers/loadlocalauthv2-failed-to-credentialsdatafrombase64/84597)

>> Hello folks.
>> I've deployed a Connect Server and the `connect-api` is having the error:
>>
>> ```
>> {"log_message":"(E) Server: (unable to get credentials and initialize API, retrying in 30s), Wrapped: (failed to FindCredentialsUniqueKey), Wrapped: (failed to loadCredentialsFile), Wrapped: (LoadLocalAuthV2 failed to credentialsDataFromBase64), illegal base64 data at input byte 0","timestamp":"2022-07-05T19:55:41.177187902Z","level":1}
>> ```
>>
>> I'm using the `OP_SESSION` environment variable to appoint to my `1password-credentials.json` file. (https://developer.1password.com/docs/connect/connect-server-configuration#environment-variables)
>>
>> Any ideas about what must be causing this error?
>>
>> Thank you!
>
> Posting my findings in case anyone else stumbles upon the same error.
>
> **tl;dr:** the `1password-credentials.json` file needs to be **double** base64 encoded in the secret data! 🤦🏻‍♂️
>
> Context: I'm using ArgoCD to bootstrap my cluster, so I'm manually creating the secrets before deploying the Connect server. I can't use the `--from-file` argument since Argo is doing the Helm deployment, not me. The clue was in the helm chart - specifically that the credentials content was being set as `stringData`, not `data`, and was being base64 encoded. From the docs, values from the `stringData` fields get base64 encoded into the data fields when the secret is created! Jackpot!
>
> I'm struggling to figure out how to clarify this in the docs. I think what's causing the confusion is that the default secret key is `1password-credentials.json`, but it's decoded contents are expected to be encoded JSON, not raw JSON. Would that be considered a bug or is that as designed?
