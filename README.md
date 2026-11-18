# Engineering GenAI with MLflow

A single Databricks notebook that takes a GenAI app from prototype to production with MLflow 3. You build a small app that analyzes customer reviews, improve its prompt, evaluate the change, release it, deploy it as a Databricks App and monitor its quality in production.

This notebook was made for the kickoff session of the [Vadodara Databricks User Group](https://usergroups.databricks.com/vadodara-databricks-user-group/) (November 28, 2026).

- Slides: [Engineering GenAI with MLflow (Speaker Deck)](https://speakerdeck.com/taka_aki/engineering-genai-with-mlflow)
- Questions: [community discussion](https://usergroups.databricks.com/v0/forum/engineering-genai-with-mlflow-331/topic/engineering-genai-with-mlflow-vadodara-dug-nov-28-2026-questions-and-follow-up-903/)

## The question this notebook answers

When you change a prompt, how do you know the change made your app better, and how do you keep track of which prompt version produced which result?

The answer is to treat the prompt like code: version it, link every request to the version that handled it, evaluate each new version before release, and keep watching it after release.

## What you do

| Stage | Step | What you do | MLflow / Databricks feature |
|---|---|---|---|
| Development | 1 | Register the first prompt (v1) and call it from the app | Prompt Registry in Unity Catalog |
| Development | 2 | Look at what happened inside each request | Tracing |
| Development | 3 | Write v2 with an output format, rules and examples | Prompt Registry |
| Offline evaluation | 4 | Score v1 and v2 on the same test data with LLM judges | `mlflow.genai.evaluate`, `make_judge` |
| Offline evaluation | 5 | Read the results and decide whether to ship v2 | Evaluation runs, comparison view |
| Release | 6 | Move the `production` alias to v2, with no code change | Prompt aliases |
| Deployment | 7 | Deploy the app, store its traces in Unity Catalog and register the judges as monitors | Databricks Apps, traces in Unity Catalog, scorers |
| Online monitoring | 8 | Use the app and see its traces scored automatically | Production monitoring |

The same judges are used offline (Step 4) and online (Step 8).

## Requirements

- A Databricks workspace. [Databricks Free Edition](https://www.databricks.com/learn/free-edition) is enough
- Serverless compute (the default in Free Edition)
- The model `databricks-meta-llama-3-3-70b-instruct`. You can check the available models on the **AI Gateway** page and change `MODEL_ENDPOINT` in Step 0

## How to run

1. Download `genai_prototype_to_production.ipynb` from this repository
2. In your workspace, click **Workspace**, open your home folder, and choose **Import** to import the file
3. Open the notebook, connect it to serverless compute and click **Run all**

Running the whole notebook takes a while, mainly for evaluation (Step 4) and app deployment (Step 7). The monitors score new traces about every 15 minutes, so the results in Step 8 appear 15 to 30 minutes after you use the app.

You can run the notebook again from the top. Step 0 deletes the development experiment and the prompt from the previous run (`START_FRESH = True`), so the prompt versions start again from v1. The production experiment, the app and the monitors are kept.

## What the notebook creates

| What | Name |
|---|---|
| Schema (with the review table and the prompt) | `workspace.genai_demo` |
| Prompt | `workspace.genai_demo.review_analyzer` |
| Development experiment (traces, evaluation runs) | `/Users/<you>/genai_prototype_to_production` |
| Production experiment (the app's traces, monitors) | `/Users/<you>/genai_review_app_production` |
| Trace tables for the production experiment | `workspace.genai_demo.review_app_otel_spans`, `_logs`, `_metrics`, `_annotations` |
| App code | `/Workspace/Users/<you>/genai_review_app` |
| Databricks App | `genai-review-analyzer` |

## Cleaning up

The last two cells of the notebook clean up. Both are switched off by default, so **Run all** does not delete anything.

| Cell | What it does | How to run it |
|---|---|---|
| Stop the monitors | Stops and deletes the two monitors. Everything else stays | Set `RUN_STOP_MONITORS = True` |
| Full reset | Deletes everything in the table above | Set `RUN_FULL_RESET = True` |

The monitors keep scoring new traces until you stop them. On Free Edition, you can have up to 3 apps, and an app stops automatically 24 hours after it starts.

## Notes

- **Traces in Unity Catalog:** The production experiment stores its traces in Unity Catalog tables, so you can query them with SQL. On Free Edition, this is also required: a Databricks App can't upload trace data to the experiment's default storage. The app's service principal needs `MODIFY` and `SELECT` on the four trace tables. The notebook grants them in Step 7.
- **Permissions for the app:** Prompt Registry is in Beta. During the Beta, loading a prompt needs `CREATE FUNCTION`, `EXECUTE` and `MANAGE` on the schema, even for read-only use. `MANAGE` is a strong privilege, so in a real project, keep prompts in a schema of their own.
- **LLM output varies:** Your evaluation results may not match the numbers in the session exactly.

## Questions

Post your questions in the [community discussion](https://usergroups.databricks.com/v0/forum/engineering-genai-with-mlflow-331/topic/engineering-genai-with-mlflow-vadodara-dug-nov-28-2026-questions-and-follow-up-903/) on the Vadodara Databricks User Group page.

## Learn more

- [MLflow 3 for GenAI on Databricks](https://docs.databricks.com/aws/en/mlflow3/genai/)
- [Prompt Registry](https://docs.databricks.com/aws/en/mlflow3/genai/prompt-version-mgmt/prompt-registry/)
- [Tracing](https://docs.databricks.com/aws/en/mlflow3/genai/tracing/)
- [Evaluation and monitoring](https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/)
- [Production monitoring](https://docs.databricks.com/aws/en/mlflow3/genai/eval-monitor/production-monitoring)
- [Store MLflow traces in Unity Catalog](https://docs.databricks.com/aws/en/mlflow3/genai/tracing/trace-unity-catalog)
- [Databricks Apps](https://docs.databricks.com/aws/en/dev-tools/databricks-apps)
- [MLflow](https://mlflow.org/)

## 日本語の方へ

このリポジトリは、MLflow 3 を使って生成AIアプリをプロトタイプから本番まで進めるノートブックです。プロンプトのバージョン管理、トレース、オフライン評価、リリース、Databricks Apps へのデプロイ、本番モニタリングまでを1本で体験できます。ノートブックは英語です。Databricks Free Edition で動きます。

- [Databricks における MLflow 3 (生成AI)](https://docs.databricks.com/aws/ja/mlflow3/genai/)
- [プロンプトレジストリ](https://docs.databricks.com/aws/ja/mlflow3/genai/prompt-version-mgmt/prompt-registry/)
- [トレーシング](https://docs.databricks.com/aws/ja/mlflow3/genai/tracing/)
- [評価とモニタリング](https://docs.databricks.com/aws/ja/mlflow3/genai/eval-monitor/)
- [Databricks Apps](https://docs.databricks.com/aws/ja/dev-tools/databricks-apps)

## Author

Takaaki Yayoi, Databricks Japan ([LinkedIn](https://www.linkedin.com/in/takaaki-yayoi/), [Qiita](https://qiita.com/taka_yayoi), [X](https://x.com/taka_aki))
