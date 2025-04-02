---
title: Azure deploy center build typescript project 發生'tsc' is not recognized as an internal or external command
tags: [devops, javascript, azure, typescript, node]

---

# Azure deploy center build typescript project 發生'tsc' is not recognized as an internal or external command

最近將  Azure bot( typescript 版本) 透過 deploy center 提供的 deploy-continuous 服務去作部署 

> Azure bot : [官方教學](https://docs.microsoft.com/zh-tw/azure/bot-service/bot-service-quickstart-create-bot?view=azure-bot-service-4.0&tabs=csharp%2Cvs)

## 先來看 Azure 官網的部署說明

| 執行階段 | 根目錄檔案 |
| --- | --- |
| ASP.NET (僅限 Windows) | **.sln*、**.csproj* 或 *default.aspx* |
| ASP.NET Core | **.sln* 或 **.csproj* |
| PHP | *index.php* |
| Ruby (僅限 Linux) | *Gemfile* |
| Node.js | 含啟動指令碼的 *server.js*、*app.js*, 或 *package.json* |
| Python | **. .py*、 *requirements.txt*或 *runtime.txt* |
| HTML | *default.htm*、*default.html*、*default.asp*、*index.htm*、*index.html* 或 *iisstart.htm* |
| WebJobsWebJobs | *job_name > /run. <持續 Webjob App_Data/jobs/continuous 下的擴充 >*功能，或針對觸發的 webjob < 。 > 如需詳細資訊，請參閱 [Kudu WebJobs 文件](https://github.com/projectkudu/kudu/wiki/WebJobs)。 |
| 函式 | 請參閱 [Azure Functions 的持續部署](https://docs.microsoft.com/zh-tw/azure/azure-functions/functions-continuous-deployment#requirements-for-continuous-deployment)。 |

> 有Node.js 

## 實際部署後

部署請參考 : [官方文件](['tsc' is not recognized as an internal or external command](https://docs.microsoft.com/zh-tw/azure/app-service/deploy-continuous-deployment?tabs=github))

之後把 souce code commit & push 到 git 上 會在Azure console 發生
```bash=
'tsc' is not recognized as an internal or external command
```

## 解決辦法

編輯 專案中的  ``` package.json ``` 將 devDependencies 的 設定 都複製加到 dependencies

example :

```json=
    "dependencies": {
        "botbuilder": "~4.15.0",
        "dotenv": "~8.2.0",
        "replace": "~1.2.0",
        "restify": "~8.5.1",
        "@types/restify": "8.4.2",
        "nodemon": "^2.0.4",
        "tslint": "^6.1.2",
        "typescript": "^3.9.2"		
    },
    "devDependencies": {
        "@types/restify": "8.4.2",
        "nodemon": "^2.0.4",
        "tslint": "^6.1.2",
        "typescript": "^3.9.2"
    }
```

之後再次部屬就可以成功了

enjoy!!

###### tags: `devops` `azure` `typescript` `node` `javascript`