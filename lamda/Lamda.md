# 📘 AWS Lambda Hands-on Lab Workbook

This workbook provides a **step-by-step guide** for mastering AWS Lambda through 16 progressively advanced labs.
Each lab contains:

* Objective
* Setup instructions
* Code/configuration snippets
* Verification steps

---

## ✅ Prerequisites

* AWS account with admin/sandbox privileges.
* Installed tools:

  * AWS CLI (`aws configure` completed)
  * AWS SAM CLI
  * Docker (for local testing & container builds)
  * Node.js or Python (runtimes)
* IAM user/role with `AdministratorAccess` (for learning only).

---

# 🧪 Lab 1 — Hello World (Console)

**Goal:** Create a Lambda function and test it.

1. Console → **Lambda** → *Create function* → Author from scratch.

   * Name: `hello-world`
   * Runtime: Node.js 18.x (or Python 3.11)
   * Role: *Create new role with basic Lambda permissions*.
2. Paste code:

```js
exports.handler = async (event) => ({ statusCode: 200, body: "Hello from Lambda" });
```

3. Deploy → Test (use sample event).

✅ Verify: Response in console + CloudWatch log created.

---

# 🧪 Lab 2 — Deploy via CLI (zip)

**Goal:** Deploy Lambda with AWS CLI.

```bash
mkdir lambda-zip && cd lambda-zip
cat > index.js <<'JS'
exports.handler = async (event) => ({ statusCode: 200, body: "CLI Hello" });
JS
zip function.zip index.js

aws lambda create-function \
  --function-name cli-hello \
  --runtime nodejs18.x \
  --role arn:aws:iam::ACCOUNT_ID:role/lambda-exec-role \
  --handler index.handler \
  --zip-file fileb://function.zip

aws lambda invoke --function-name cli-hello out.json
cat out.json
```

✅ Verify: Output contains `CLI Hello`.

---

# 🧪 Lab 3 — SAM: Local Dev & Deploy

**Goal:** Build/test locally, deploy with SAM.

```bash
sam init --runtime python3.11 --name sam-hello --app-template hello-world
cd sam-hello

sam build
sam local invoke HelloWorldFunction --event events/event.json
sam local start-api   # http://127.0.0.1:3000/hello

sam deploy --guided
```

✅ Verify: API Gateway endpoint returns JSON.

---

# 🧪 Lab 4 — API Gateway Trigger

**Goal:** Expose Lambda as HTTP endpoint.

* Console: Lambda → Add trigger → API Gateway → HTTP API.
* Test with curl:

```bash
curl https://<api-id>.execute-api.<region>.amazonaws.com/hello
```

✅ Verify: Response from Lambda.

---

# 🧪 Lab 5 — S3 Trigger

**Goal:** Process objects uploaded to S3.

1. Create S3 bucket.
2. Lambda → Add trigger → S3 → ObjectCreated.
3. Handler (Python):

```python
import boto3
def lambda_handler(event, context):
    s3 = boto3.client('s3')
    rec = event['Records'][0]
    bucket, key = rec['s3']['bucket']['name'], rec['s3']['object']['key']
    obj = s3.get_object(Bucket=bucket, Key=key)
    print("Object size:", obj['ContentLength'])
    return {"status": "ok"}
```

✅ Verify: Upload file → CloudWatch logs show size.

---

# 🧪 Lab 6 — DynamoDB Streams

**Goal:** Process change events.

1. Create DynamoDB table with Streams enabled.
2. Lambda → Add trigger → DynamoDB Stream.
3. Logs incoming changes.

✅ Verify: Insert item → log entry generated.

---

# 🧪 Lab 7 — SNS & SQS Triggers + DLQ

* Subscribe Lambda to SNS topic.
* Configure Lambda with SQS event source.
* Set DLQ in function config.

✅ Verify: Publish SNS message or push SQS message → Lambda invoked. Failed events land in DLQ.

---

# 🧪 Lab 8 — Layers & Secrets

**Goal:** Share libs + secure secrets.

* Create Lambda Layer with `node_modules` or Python packages.
* Add environment variable `MY_SECRET_ARN`.
* Fetch secret:

```python
import boto3, os
sm = boto3.client('secretsmanager')
secret = sm.get_secret_value(SecretId=os.environ['MY_SECRET_ARN'])['SecretString']
```

✅ Verify: Secret retrieved securely.

---

# 🧪 Lab 9 — VPC Access

**Goal:** Connect Lambda to RDS.

* Configure VPC, subnets, SG.
* Attach Lambda to private subnets.
* Add IAM policy: `AWSLambdaVPCAccessExecutionRole`.

✅ Verify: Lambda can query RDS instance.

---

# 🧪 Lab 10 — Monitoring & Tracing

* Enable CloudWatch Logs (default).
* Enable X-Ray: `Tracing: Active`.
* Query logs:

```sql
fields @timestamp, @message | filter @message like /Error/ | limit 20
```

✅ Verify: View traces in X-Ray.

---

# 🧪 Lab 11 — Performance & Cost

* Increase memory (boosts CPU).
* Adjust timeout.
* Enable Provisioned Concurrency.

✅ Verify: Reduced cold start latency.

---

# 🧪 Lab 12 — Container Image Lambda

1. Dockerfile:

```dockerfile
FROM public.ecr.aws/lambda/python:3.11
COPY app.py ./
CMD ["app.lambda_handler"]
```

2. Build & push to ECR.
3. Create Lambda from image.

✅ Verify: Function runs from container.

---

# 🧪 Lab 13 — Step Functions Orchestration

* Create multiple Lambdas.
* Define state machine with `Task` → Lambda ARNs.

✅ Verify: Execution graph runs steps.

---

# 🧪 Lab 14 — CI/CD with GitHub Actions (SAM)

```yaml
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-region: us-east-1
        role-to-assume: arn:aws:iam::123456:role/GitHubDeployRole
    - run: |
        sam build
        sam package --s3-bucket $S3_BUCKET --output-template-file packaged.yml
        sam deploy --template-file packaged.yml --stack-name my-stack --capabilities CAPABILITY_IAM
```

✅ Verify: Deployment runs automatically on push.

---

# 🧪 Lab 15 — Security Best Practices

* Least privilege IAM policies.
* No plaintext secrets → use Secrets Manager / KMS.
* Restrict VPC/security groups.
* Encrypt env vars.

✅ Verify: IAM access analyzer shows minimal permissions.

---

# 🧪 Lab 16 — Troubleshooting

* Timeout → increase timeout or optimize.
* AccessDenied → missing IAM permissions.
* Cold starts → provisioned concurrency or SnapStart (Java).
* Logs missing → check role has `AWSLambdaBasicExecutionRole`.

✅ Verify: Errors resolved systematically.

---

# 📌 CLI Cheatsheet

```bash
aws lambda invoke --function-name myfn out.json
aws lambda update-function-code --function-name myfn --zip-file fileb://function.zip
aws lambda put-provisioned-concurrency-config --function-name myfn --qualifier '$LATEST' --provisioned-concurrent-executions 5
```

---

# 🎯 Learning Milestones

By completing these labs you will:

* Build, test, and deploy Lambda functions (console, CLI, SAM).
* Trigger Lambda from API Gateway, S3, DynamoDB, SQS, SNS.
* Monitor with CloudWatch + X-Ray.
* Secure with IAM, VPC, Secrets Manager.
* Optimize for performance & cost.
* Deploy with CI/CD pipelines.

---

✅ End of Workbook — You are now ready for **AWS Lambda interview and production scenarios**.
