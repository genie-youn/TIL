Debian에서 Puppeteer 실행시 크래시가 발생한다면 다음을 시도해보자.

#### "Error while loading shared libraries: libnss3.so" 등과 같은 에러가 발생할 경우
```bash
sudo apt install libnss3 libatk1.0-0 libatk-bridge2.0-0 libcups2 libgbm1 lib asound2 libpangocairo-1.0-0 libxss1 libgtk-3-0
```


#### Received signal 6

puppeteer 실행시 --no-sandbox, --disable-setuid-sandbox 옵션 추가

```javascript
await puppeteer.launch({args: ['--no-sandbox', '--disable-setuid-sandbox']})
```

https://github.com/puppeteer/puppeteer/issues/290


CentOS에 Puppeteer 실행시 크래시가 발생한다면 다음을 시도해본다.

####  Error: Failed to launch the browser process! : error while loading shared libraries: libatk-1.0.so.0: cannot open shared object file: No such file or directory
```
sudo yum install -y chromium 
```

https://github.com/puppeteer/puppeteer/issues/5361
