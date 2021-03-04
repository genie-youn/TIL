# PM2에서 dotenv 주입하기

기존에 express를 node로 npm 통해서 바로 실행시킬땐 다음과 같이 사용했었음

package.json
```json
"script": {
  "start:development": "node -r dotenv/config ./app.js dotenv_config_path=./environment/.env.development",
  "start:production": "node -r dotenv/config ./app.js dotenv_config_path=./environment/.env.production",
}
```

동일한 접근으로 pm2 도 다음과 같이 실행시키려 했는데

ecosystem.config.js
```javascript
module.exports = {
  apps : [{
    name : 'app',
    script : './app.js',
    node_args : '-r dotenv/config dotenv_config_path=./environment/.env.development',
    ...
  }],
}
```

곧죽어도 설정이 안됨. 결국 `ecosystem.config.js` 에선 `NODE_ENV` 만 주입해주고 앱 내에서 이 값에 따라 dotenv 설정 파일을 불러오도록 변경

ecosystem.config.js
```javascript
module.exports = {
  apps : [{
    name : 'app',
    script : './app.js',
    env: {
      "NODE_ENV": "development",
    }
    ...
  }],
}
```

app.js
```javascript
const path = require('path');
require('dotenv').config({ path: path.join(__dirname, `/environment/.env.${NODE_ENV}``) });
```
