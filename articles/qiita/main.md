This page is for qiita sites of articles.

export html of the qiita main page and operate below
* execute this command by git bash or cmd.
```
curl -OH 'Authorization: Bearer unkoauthentification_token' 'https://qiita.com/api/v2/authenticated_user/items?page=numbers&per_page=20'  --insecure
```
※NOTE1:**unkoauthentification_token** is your qiita personal settings. 
※NOTE2:page=**numbers**&per_page=20 is vary. please change **numbers** if nessesary.
* substitute "," to "\n"
* this command "grep keyword"
"https://qiita.com/himitsuunkoman/items"

then it seems to be found items/~ some URLs...

```
curl -O https://qiita.com/himitsuunkoman/items/hogeeeeeeee1.md --ssl-no-revoke
curl -O https://qiita.com/himitsuunkoman/items/hogeeeeeeee2.md --ssl-no-revoke
...
curl -O https://qiita.com/himitsuunkoman/items/hogeeeeeeeen.md --ssl-no-revoke
```
