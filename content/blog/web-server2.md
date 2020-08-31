---
title: 웹 서버 구축하기 (2) nodejs로 서버 로직 만들기
date: 2020-06-02 12:21:13
category: 'web-server'
thumbnail: './images/mac.jpg'
draft: false
---
![](./images/mac.jpg)
<div style="text-align: center;">
    <span style="font-size:11px; color:grey">
        이 정적 페이지는 PC 버전에 최적화되어 있습니다.  
    </span>
</div>

---
이전 글에서 웹 서버가 기본적으로 필요로 하는 소프트웨어의 설정 및 기본 설정을 완료하였다.  
이번에는 직접 서버의 로직을 만들겠다.

## 꼭 필요한 npm 패키지 라이브러리 설치

node로 서버를 띄우기 위해서 필요한 최소한의 라이브러리는
- babel (트랜스파일러)
- express (미들웨어)

## 선택적인 라이브러리 설치

- webpack (모듈 번들러)
- pm2 (프로세스매니저)
- swagger (GUI api Docs)

다른 프로젝트에서도 사용할 일이 많으므로 전역 설치한다.  
```bash
npm install -g babel-cli
npm install -g babel-preset-env
npm install -g express-generator
npm install -g pm2
npm install -g webpack
```

## express 프로젝트 생성

```bash
express 프로젝트이름
cd 프로젝트이름
npm install
```

## routes 작성하기

express 프로젝트는 ./bin/www에서
Listening, port 설정 등을 해준다.  
따라서 route를 작성하고 app.js에 아래와 같이 추가만 하면 된다.  
```javascript
// Import Route Lists
import index from './routes/index';
import update from './routes/update';
import login from './routes/login';
import add_gold from './routes/add_gold';
import add_cash from './routes/add_cash';

// Add Route Lists
app.use('/', index);
app.use('/update', update);
app.use('/login', login);
app.use('/add_gold', add_gold);
app.use('/add_cash', add_cash);
app.use('/create_nickname', create_nickname);
```

## DB 연동하기

mysql2/promise 패키지 모듈은 node와 mysql이 연동하기 가장 쉽게 잘 되어 있는 모듈이다.  
별도의 Manager 폴더를 만들고 아래와 같은 DB manager를 구성한다.  

```javascript
const mysql      = require('mysql2/promise');
// const config = require('../config/db_config_local.json');
const config = require('../config/db_config_real.json');

const pool = mysql.createPool(config);

export async function getConnection() {
    try {
        const connection = await pool.getConnection();
        return connection;
    } catch (err) {
        console.log(err);
        return false;
    }
}

export async function executeQuery(connection, query, param) {
    try {
        try {
            await connection.beginTransaction();
        } catch(err) {
            console.log(err))
            var connection = await pool.getConnection();
        } finally {
            const [rows] = await connection.query(query, param);
            await connection.commit();
            return rows;
        }
    } catch (err) {
        console.log(err);
        await connection.rollback();
        await connection.release();
        return false;
    } finally {
        await connection.release();
    }
}
```
DB config는 필요에 따라 connectionLimit 혹은 waitForConnections 정도만 추가해주면 좋다.  
```
{
    "host": "127.0.0.1",
    "user": "root",
    "password": "1234",
    "database": "root",
    "port": 3306,
    "connectionLimit": 4
}
```

## 파일 내려주기
 
서버에서 필요에 따라 데이터 파일 등을 내려 줘야할때가 있다.  
위에서 DB manager를 만든 것처럼 FileManager를 만들어 아래와 같은 함수를 추가하자.  
```javascript
const fs = require('fs');
const path = require('path');
const { promisify } = require('util');
const readFileAsync = promisify(fs.readFile);
const writeFileSync = promisify(fs.writeFile);
const readDir = promisify(fs.readdir);

export async function GetAllFileData() {
    try {
        const dataDir = path.join(__dirname, '../data');
        var files = await readDir(dataDir);
        var fileData = {};
        for (var i in files) {
            var file = files[i]
            fileData[file] = await readFileAsync(path.join(dataDir, file));    
        }
        return fileData;
    } catch (err) {
        console.log(err);
    }
}
```
클라에게 매번 데이터 파일을 내려줄 필요가 없는 경우 cryto 모듈의 md5를 활용하여 hash 값이 다를 때만 내려주도록 하자.
```javascript
const crypto = require('crypto');

export async function GetMd5Data() {
    var fileData = await GetAllFileData();
    var md5 = crypto.createHash('md5');
    var data_md5 = md5.update(JSON.stringify(fileData)).digest('hex');
    console.log(data_md5);
    return data_md5;
}
```

## 구글 API와 연동하기

필요한 경우 엑셀 시트 등의 구글 Docs를 연동하여 데이터파일을 만들 수 있다.  
아래의 함수는 시트에서 특정 행, 열에서 데이터를 가져와 파일로 새로 쓰는 함수이다.  
```javascript
const { GoogleSpreadsheet } = require('google-spreadsheet');
const creds = require("../config/key.json");
const doc = new GoogleSpreadsheet("1OoY3F9JAY1Lu34jTVBg4jDqpKESaoj0rHRij3ADMLxk");

export async function CreateFile() {
	await doc.useServiceAccountAuth(creds);
	console.log("start create file");
	await doc.loadInfo();

	// C10부터 데이터가 존재한다.
	for (var i = 2; i < doc.sheetCount; i++) 
	{
		var sheet = doc.sheetsByIndex[i];
		console.log("Target Table : " + sheet.title);
		if (sheet.title == "skill")
			await sheet.loadCells('C10:AL600');
		else 
			await sheet.loadCells('C10:Z600');
		var startRowIndex = 9;
		var startColumnIndex = 2;
		var data = '';
	
		while (1) 
		{
			var start = sheet.getCell(startRowIndex, startColumnIndex);
			if (start.value == null) 
			{
				if (startColumnIndex == 2) // end of data
					break;
				startRowIndex++;
				startColumnIndex = 2;
				data += '\r\n';
			}
			else 
			{
				data += start.value + ' ';
				startColumnIndex++;
			}
		}
		await writeFileSync('./data/'+sheet.title, data)
	}
}
```




