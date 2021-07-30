---
title: "docker-compose up 時にでたエラーの解決"
emoji: ":astonished:"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Docker", ]
published: true
---

# エラー内容

```
ERROR: for docker_rails_web_1  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)

ERROR: for web  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)
ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.
If you encounter this issue regularly because of slow network conditions, consider setting COMPOSE_HTTP_TIMEOUT to a higher value (current value: 60).
```

## 考えられる原因
よくわからないが、Docker for Macの原因らしい、、

https://github.com/docker/compose/issues/5620
https://github.com/crowi/crowi/issues/308

# 解決策
Docker Desktopの再起動で解決 :raised_back_of_hand:

Restartで再起動！！

![](https://storage.googleapis.com/zenn-user-upload/63b89d9d7d068111effca177.png)

# 参考
https://qiita.com/Kattrxvjxxde/items/cfc689b7b204ba3b1202