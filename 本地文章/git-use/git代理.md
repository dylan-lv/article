### 设置代理

```bash
//http || https
git config --global http.proxy 127.0.0.1:1080
git config --global https.proxy 127.0.0.1:1080

//sock5代理
git config --global http.proxy socks5 127.0.0.1:1080
git config --global http.proxy socks5 127.0.0.1:1080
```



### 查看代理

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```



### 取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```





